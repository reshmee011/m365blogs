---
title: "Powerautomate_dateactions"
date: 2024-03-28T05:24:43Z
draft: true
---

DATE TIME

. Add to time
. Convert time zone
. Current time
. Get future time
. Get past time
. Subtract from time
. Time units range all the way from months, down to seconds.
. Example: Add 2 days to the current time, to create a due date

ADD TO TIME

. The AddDays expression is commonly used to achieve this same
concept. Add to a date/time field.
. The Subtract from time action does the opposite.
· Get future time is even quicker, always relative to now, in UTC.

EXAMPLE: TASKS DUE WITHIN 2 DAYS

· Trigger: Recurrence
. Action: Current time - Current time is always UTC
. Action: Convert time zone - use your own time zone
. Action: Add to time - 2 days
. Action: Get items
. Status not completed
· Due within 2 days

SCHEDULE

· Delay
· Waits a certain number of days,
hours minutes, etc. Then the
workflow continues.
· Delay Until
· Waits until a specific timestamp
value.

CONDITION IF EMPTY
· There is no option for checking if a column is
empty. The trick is to use a variable.
· Example:
. Check to see if the column Date Completed is
blank.
· After a task is modified or created, initialize
and set a variable called varDateCompleted.
. In the condition, use the variable, is equal to,
and leave value blank.


MORE USEFUL EXPRESSIONS

. FormatDateTime - Specify how a date should look
formatDateTime(SelectAfield,'MM-dd-yyyy')

· DayOfWeek - lets you know what day of the week that a specific date
was on
dayOfWeek(triggerBody()?['Created'])
. In this example, it lets me know the day of the week that this item was
created.