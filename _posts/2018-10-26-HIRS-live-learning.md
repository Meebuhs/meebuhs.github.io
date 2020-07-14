---
layout: post
title: HIRS Live Learning
description: A digital alternative to paper activity sheets for use by school groups visiting UQ's Heron Island Research Station.
category: completed
permalink: /projects/hirs-live-learning
images:
    cover: /assets/images/hirs-live-learning/cover.jpg
    top: /assets/images/hirs-live-learning/cover.jpg
tags: [react, electron, yarn, javascript, progressive web app]
---

Every year thousands of high school students visit the research station on Heron Island for hands-on experience in the field of marine science. Over their time on the island, these students fill out numerous activity sheets to record their experiment results and collected data. These sheets were frustrating to use for the students as they were cumbersome to handle in the wind and would often get wet while the students were collecting data in the ocean. It also required a time investment from the researchers to digitise the student's data.

This project allows the researchers to create digital activity sheets, and allows the students to use iPads to complete them around the island and submit them. It was created collaboratively by a multi-disciplinary team of 8 students as part of the DECO3801 course at the University of Queensland. 

## Activity Creator

<div class="clickable-image"><a href="/assets/images/hirs-live-learning/activity-creator.png">
    <img src="/assets/images/blank.png" alt="HIRS Live Learning App - Activity Creator" data-echo="/assets/images/hirs-live-learning/activity-creator.png" />
</a></div>

This part of the project was a portable program which ran on the island's desktop computers and allowed the staff to create the activity sheets. It features a simple user interface where elements are added to the sheet with the click of a button and then rearranged by dragging and dropping. This eliminates the need for the researcher to spend time layout out and typesetting the activity sheet. The activity sheets are made up of any number of input elements, defining the input required from the students.

The supported input types are:
- Text - A simple text box, either single or multi line, where a student can input a name or a description.
- Counter - A simple counter which the students can increment.
- Location - A button which attaches the GPS coordinates of their current location, useful for noting where a specimen was found. 
- Photo - Allows the student to use the iPad camera to attach a photo to their data
- Checkbox - Single or multi, allows the students to choose from a set of options defined by the researcher.

With these input elements, and the customisable titles and descriptions set by the researcher, it is possible to create detailed activity sheets with minimal time investment or effort.

Once completed the activity sheets are saved to a JSON file which is read by the iPad app. I was personally responsible for designing and writing this activity creator in its entirety using react, yarn and electron.

## Live Learning App

This part of the project was a progressive web app which ran on the island's iPads. The app automatically populates with the activity sheets created in the activity creator and allows each group of students to create a profile, select an activity sheet and fill them out. Data which they input to the sheets is automatically stored as Excel-readable CSVs on the island desktops as changes are made to them. If the students are not within range of wifi, which happens quite often on the island, the data is cached on the device and uploaded when the connection is restored.

This system means that researchers are never required to pull data off the iPads as all data is already digitised and available on the island desktops.

I was responsible for writing the javascript functionalty to parse the JSON activity sheets and correctly display them in the app.