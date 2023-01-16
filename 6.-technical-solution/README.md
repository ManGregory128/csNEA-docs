# 6. Technical Solution

### 6.1 – System Overview

The solution will be programmed in C# and will utilize Windows Forms and Xamarin Forms to create its graphical user interface. Administrators may log in using the pre-set administrator user in the “users” table of the database to set the system up for more users. They must set up the school’s lessons, add groups and more users, and finally add the students. Teachers can then use the mobile application to take daily registers that administrators and secretaries can view through the Absences tab of the Desktop Application and can also export them as a flat file for back-up purposes. Administrators and secretaries may also want to promote students to different groups when the year changes, so they do not have to remove and import students again with updated group information. All the tables required to store the necessary information will be found in the Database “aradippou5” named after the school. This means the database can be replicated on the same or different servers to allow for other schools to independently use the same solution.

Once the mobile application is launched, it will check for any stored user login information and present the appropriate screen – either the log-in screen which lets users marked as teachers use the application, or the main page which is tabbed. Logged in users can check the feed for new posts through the feed tab, update their password or log out using the settings tab or just use the home tab to take the register. The application will have calculated the current date beforehand, and will allow the teacher to choose which period they want to record information for – periods will be annotated with the lessons matching the teaching information provided by administrators when they upload schedules through the desktop application. Once a period is selected, the application will determine whether attendance information for that date, period and group have been recorded yet. If they have, teachers can update their register. On the other hand, only the students required will be fetched and new attendance records will be recorded if no attendance information already exists in the Attendances table.

Teachers take or update the register through a list with all the students’ names. The names are bound to a checkbox which determines the presence status of each student. Teachers simply check/uncheck to mark students as present/absent and then send the updated register to the database through a “Submit” button located at the bottom of the list.

### 6.2 – Module Structure Chart

<figure><img src="../.gitbook/assets/module structure chart.png" alt=""><figcaption></figcaption></figure>

### 6.3 – Implementation – Program Code

This section of the chapter is split into two sub-sections, both of which can be found separately:

#### ****[**6.3.1. Implementing the Desktop Application**](implementing-the-desktop-application.md)****

**6.3.2. Implementing the Mobile Application**
