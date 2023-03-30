---
layout: post
title: Incident Management System Part 1 - Incident Report Form
excerpt_separator:  <!--more-->
---
## In this post
- [Project Overview](#project-overview)
- [The Data Model](#data-model)
- [Incident Report Form](#incident-report-form)
    - [The easy bit: Text input, Datepicker](#easy-bit)
    - [The not so easy bit: User Lookup](#not-so-bit)
    - [The roundabout bit: File attachments](#roundabout-bit)

<br>
<section id="project-overview"><h2>Project Overview</h2></section>

This solution is designed to streamline the process of managing and resolving incidents within an organisation. The system consists of three components that work together to enable an efficient and effective incident management process.

1. **Incident Report Form** Staff report incidents by filling out a form using a Power App canvas app. This form allows them to provide all relevant details about the incident, such as location, and action taken.

2. **Incidents Dashboard** The Estates team uses a dashboard canvas app to follow up and take action on the reported incidents. This dashboard allows them to see all the incidents reported and to prioritise their response based on the severity of the incident. They can then assign the incident to a team member and track the progress of its resolution.

2. **Incident Management Dashboard** Managers can keep track of all incidents and assignments using a model driven app. This app allows them to view all the incidents reported, their current status, and the assigned team member responsible for resolving the incident. They can use this information to monitor the performance of the Estates team and to ensure that all incidents are resolved in a timely and effective manner.

All the data for this system is stored in Dataverse, which serves as the central repository for all incident-related information. This allows for easy tracking and analysis of incidents and their resolution, as well as the ability to generate reports and metrics for performance management purposes.

<br>
<section id="data-model"><h2>The Data Model</h2></section>

[Incident Management System](https://drawsql.app/teams/gmvflow/diagrams/contoso-estates)
![Schema](/docs/images/data-model.png)

<br>
In Dataverse, this translates as:

![Dataverse Tables](/docs/images/dataverse-tables.png)

All tables, with the exception of `Users`, are custom tables. `Users` is a default table added to the environment upon creating a User lookup.

Where values are constant, e.g. `priorities`, `types`, etc., I have created `Choice` type columns, that is, in place of lookup tables.

<br>
<section id="incident-report-form"><h2>Incident Report Form</h2></section>

<section id="easy-bit"><h3>The easy bit: Text input, Datepicker, etc.</h3></section>

Starting off with the simple data types in the `Incident` table highlighted in yellow below. (Columns highlighted in grey do not feature in the Incident Report Form.)

![Simple columns](/docs/images/simple-columns.png)

<br>
In the form UI, laid out as follows:

![Simple ui](/docs/images/simple-ui.png)

1. Date picker `dteDateOfIncident`
2. Text input `txtLocation` and `txtActionTaken`
3. Rich text editor `rchDescription`

<br>

Patching is fairly straightforward:
```
Patch(
    Incidents,
    Defaults(Incidents),
    {
        'Date Of Incident': dteDateOfIncident.SelectedDate,
        Location: txtLocation.Text,
        'Action Taken': txtActionTaken.Text,
        Description: rchDescription.HtmlText
    }
)
```
<br>
<section id="not-so-bit"><h3>The not so easy bit: User Lookup</h3></section>

The `Reported By` column is a lookup column which connects to the default `Users` table. 

![User Lookup](/docs/images/user-lookup.png)
![Reported by details](/docs/images/reported-by-lookup.png){: width="300" }

<br>
This field is prepopulated with its value set to the current user, so a simple Text Input can be used to display the value and the `DisplayMode` set to `Disabled`.

![Reported By](/docs/images/reported-by.png)

```
// Default propery
User().FullName

// DisplayMode property
DisplayMode.Disabled
```
<br>
When patching a lookup column, we use the `Lookup()` function to look things up in the related table.

```
Patch(
    Incidents,
    Defaults(Incidents),
    {
        'Reported By': Lookup(
            Users,
            'Full Name' = User().FullName
        )
    }
)
```
<br>

Putting it all together:
```
Patch(
    Incidents,
    Defaults(Incidents),
    {   
        'Date Of Incident': dteDateOfIncident.SelectedDate,
        Location: txtLocation.Text,
        'Action Taken': txtActionTaken.Text,
        Description: rchDescription.HtmlText,
        'Reported By': Lookup(
            Users,
            'Full Name' = User().FullName
        )
    }
)
```

<br>
<section id="roundabout-bit"><h3>The roundabout bit: File attachments</h3></section>

To handle file attachments in Dataverse, I created a separate table `Supporting Files` with a File-type column and a Lookup-type column which links to the main table, i.e. `Incidents`.

![Supporting Files table](/docs/images/supporting-files.png)

All files are stored in the `FileAttachment` entity. When creating a File-type column, the column automatically links to the `FileAttachment` entity creating a Many-to-one relationship.

![FileAttachment](/docs/images/fileattachment-entity.png)

<br>
#### The Attachments control
The `Attachments` control is not readily available from the UI components list and has to be cut or copied from inside a `Form` control.

![Form control](/docs/images/form-control.png)

<br>
A File-type column in Dataverse requires an object with 2 keys: 
1. `FileName`
2. `Value`

The `Attachments` control is a list of attachments (set `MaxAttachments` property as appropriate) with the corresponding "Name" and "Value" of each file.

File attachments are handled in a separate patch call to the corresponding table, i.e. `Supporting Files`.

When patching, we have to loop over each attachment and patch each item individually thereby creating a discrete record each time. We then map the "Name" and "Value" of each file to the required keys.

```
ForAll(
    attachmentIncidentFile.Attachments,
    Patch(
        'Supporting Files', 
        Defaults('Supporting Files'),
        'Incident File': {
            FileName: ThisRecord.Name,
            Value: ThisRecord.Value
        }
    )
)
```

<br>
The image below shows 2 attachment records created in the same form submission. 

![Attachment record](/docs/images/attachment.png)

<br>
`Incident Lookup` is a lookup column which links to the `Incidents` table in a Many-to-one relationship. This column expects the last patched record (similar to `lastSubmit` when using a `Form` control) in the `Incidents` table. 

To capture the last patched record we have to store the patched record in a variable. 

Previously:
```
Patch(
    Incidents,
    Defaults(Incidents),
    {   
        'Date Of Incident': dteDateOfIncident.SelectedDate,
        Location: txtLocation.Text,
        'Action Taken': txtActionTaken.Text,
        Description: rchDescription.HtmlText,
        'Reported By': Lookup(
            Users,
            'Full Name' = User().FullName
        )
    }
)
```

Updated:
```
Set(
    varNewIncident,
    Patch(
        Incidents,
        Defaults(Incidents),
        {   
            'Date Of Incident': dteDateOfIncident.SelectedDate,
            Location: txtLocation.Text,
            'Action Taken': txtActionTaken.Text,
            Description: rchDescription.HtmlText,
            'Reported By': Lookup(
                Users,
                'Full Name' = User().FullName
            )
        }
    )
)
```

Now we can pass `varNewIncident` to the lookup field. 
```
ForAll(
    attachmentIncidentFile.Attachments,
    Patch(
        'Supporting Files', 
        Defaults('Supporting Files'),
        'Incident File': {...},
        'Incident Lookup': varNewIncident
    )
)
```

<br>

Putting it all together:
```
Set(
    varNewIncident,
    Patch(
        Incidents,
        Defaults(Incidents),
        {
            Title: myTitle,
            'Reported By': LookUp(
                Users,
                'Full Name' = User().FullName
            ),
            'Date Of Incident': datDateOfIncident.SelectedDate,
            Location: txtLocation.Text,
            Description: rchDescription.HtmlText,
            'Action Taken': txtActionTaken.Text
        }
    )
);
If(
    IsEmpty(Errors(Incidents)),

    // On Success
    ForAll(
        attachIncidentFile.Attachments,
        Patch(
            'Supporting Files',
            Defaults('Supporting Files'),
            {
                'Supporting File Name': ThisRecord.Name,
                'Incident Lookup': varNewIncident,
                'Incident File': {
                    FileName: ThisRecord.Name,
                    Value: ThisRecord.Value
                }
            }
        )
    );
    Notify(
        "Incident record successfully submitted.",
        NotificationType.Success
    ),

    //On Fail
    Notify(
        First(Errors(Incident)).Message,
        NotificationType.Error
    )
)
```

<br>
[Incident Management System Part 2 - Incidents Dashboard](#)