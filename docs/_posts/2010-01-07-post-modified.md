---
layout: post
title: "Post: Modified Date"
categories:
  - Post Formats
tags:
  - Post Formats
  - readability
  - standard
last_modified_at: 2017-03-09T13:01:27-05:00
---

This post has been updated and should show a modified date if `last_modified_at` is used in the layout.

Plugins like [**jekyll-sitemap**](https://github.com/jekyll/jekyll-feed) use this field to add a `<lastmod>` tag your `sitemap.xml`.



<table>
    <tr>
    <th>INCIDENT TABLE</th>
    </tr>
    <tr>
        <th>Display name</th>
        <th>Data type</th>
    </tr>
    <tr>
        <td>Incident</td>
        <td>Unique identifier</td>
    </tr>
    <tr>
        <td>Title (Primary name column)</td>
        <td>Single line of text</td>
    </tr>
    <tr>
        <td>Reported By</td>
        <td>Lookup (User)</td>
    </tr>
    <tr>
        <td>Date Of Incident</td>
        <td>Date only</td> 
    </tr>
    <tr>
        <td>Location</td>
        <td>Single line of text</td> 
    </tr>
    <tr>
        <td>Description</td>
        <td>Rich text</td> 
    </tr>
    <tr>
        <td>Action taken</td>
        <td>Rich text</td> 
    </tr>
    <tr>
        <td>Type</td>
        <td>Choice</td> 
    </tr>
    <tr>
        <td>Severity</td>
        <td>Choice</td> 
    </tr>
    <tr>
        <td>Status</td>
        <td>Choice</td> 
    </tr>
    <tr>
        <td>Priority</td>
        <td>Choice</td> 
    </tr>
</table>

<table>
    <tr>
    <th>SUPPORTING FILES TABLE</th>
    </tr>
    <tr>
        <th>Display name</th>
        <th>Data type</th>
    </tr>
    <tr>
        <td>Supporting Files</td>
        <td>Unique identifier</td>
    </tr>
    <tr>
        <td>Supporting File Name (Primary name column)</td>
        <td>Single line of text</td>
    </tr>
    <tr>
        <td>Incident File</td>
        <td>File</td>
    </tr>
    <tr>
        <td>Incident Lookup</td>
        <td>Lookup (Incident)</td> 
    </tr>
</table>