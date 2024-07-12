---
title: "DateTime Handling in Power Automate"
date: 2024-07-07T09:53:05+01:00
tags: ["Power Automate", "DateTime", "Data Handling"]
featured_image: '/posts/images/PowerAutomate-json-actions-body-output/get_approval_details.png'
omit_header_text: true
draft: false
---

# DateTime Handling in Power Automate

Manipulating dates and times in Power Automate is a requirement at times. This post covers a few useful scenerios.

## Add To Time

For instance, adding a specific number of months to a date retrieved from an action using AddToTime function:

> addToTime(body('Wait_for_an_approval_2')?['completionDate'], int(body('Get_file_properties')?['ReviewFrequency']?['Value']), 'Month', 'dd/MM/yyyy HH:mm')

## Format Date Time

Formatting date to a specific string format is essential otherwise update of data fields might fail because of culture differences. The snippet below formats the date into UK date format before saving it to the date field.

> formatDateTime(body('Wait_for_an_approval_2')?['completionDate'], 'dd/MM/yyyy HH:mm')

## Day of the week

The function `dayOfWeek` returns the day of the week that a specific date was on.

> dayOfWeek(triggerBody()?['Created'])

Above example returned the day of the week that this item was created.

## UTC Today's Date
 
> formatDateTime(utcNow(), 'dd/MM/yyyy HH:mm')