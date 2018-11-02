# Driver Training Courseware Site

This site provides management and content for certified corporate automobile and truck driver training courses.

These training courses are used for corporate insurance purposes, so the accuracy and reliabity of this site are of primary im
portance to clients.

The admin site implements:
- Multiple corporate clients. 
- Corporate internal structure, allowing divisions based on organizational or geographic concerns.
- Client "affiliations", grouping together corporations for convenient management of training campaigns.
- Flexible assignment of responsibility domains for administrators, based on corporation or affiliation.
- Assignment of courses to students, email notifications for students, and recording of training results.
- Comprehensive report generation of student and course status based on flexible selection criteria.
- Efficient means for selecting groups of students from among employees for campaign and course enrollments and notifi
cations.
- A simple dashboard summarizes information for administrators and allows navigation to other administrative pages.
        
The user side implements:
- Student course selection and enrollment.
- Notification and reminders of course start and required completion dates.
- Presentation of online courses with:
  - SMIL inspired FLV/SWF player with audio synchronization and multimedia controls.
  - Interactive scripted course navigation.
  - Quizzes and tests to verify student's apprehension of the course material.
  - Recording test results and course completion status.
  - Issuing certificates that students can use for insurance purposes.      
- Enrollment and scheduling for on-site behind-the-wheel training, based on course availability and location.
        
Technical aspects of the site:
- Hierarchical organization of companies, affiliations and student lists in SQL database.
- Database currently has information on ~1.3 million students and their training histories and statuses.
- Student lists, status updates and enrollment events can be imported from client-supplied spreadsheet files.
- Admin REST API allows clients to automate their employee lists, training assignments and reporting.
- CAS authentication is available for admins, students and API clients.
- Time-consuming tasks are queued and completed asynchronously from the web interface. Admins can monitor the progress
 of their queued jobs.

[Webtraning - Front Page](/projects/webtraining/webtraining_front_page_512.png)
