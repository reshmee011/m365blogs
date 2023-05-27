---
title: 'Input of type date to generate a date picker using Reagent in ClojureScript'
date: Fri, 01 Jan 2016 13:31:48 +0000
draft: false
tags: ['clojurescript', 'reagent', 'Reagent ClojureScript DatePicker', 'Uncategorized']
---

I needed a date picker to use in ClojureScript application to enter date of birth. I came across the [pikaday](https://github.com/timgilbert/cljs-pikaday)  which I tried to use but the date range was limited and could not find a solution to not limit the date range. The native HTML input control of type date can be generated in Reagent as follows `[:p "Enter Date Of Birth"] ;;using input control of type date to allow a date to be picked [:input {:type "date" :on-change (fn [e] (swap! pension-age assoc :dob (js/Date. (.-target.value e)))) }]` On the UI, the date picker ![datePicker](https://reshmeeauckloo.files.wordpress.com/2016/01/datepicker.png)