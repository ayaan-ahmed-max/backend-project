# CMP5387 Gantt chart prototype

Written project plan in order to create a formatted and cohesive gantt chart for design doc

## D2 design and submission

All seven members work on their design rows in parallel from 2 to 13 November. The deadline for handing the rows to Zeeshan and Haarisah is **12 November at noon**.

| Owner | Task | Dates |
| --- | --- | --- |
| Ayaan | Requirements, shared error format and title page | 2–13 Nov |
| Lakhan | Problem summary, stakeholders and priorities | 2–13 Nov |
| Haarisah | Entity relationship diagrams and merged database table | 2–13 Nov |
| Lee | Gantt chart, references, document assembly and sign off | 2–13 Nov |
| Zeeshan | Compile the endpoint inventory | 2–13 Nov |
| Husray | Evaluation and links between requirements and tests | 2–13 Nov |
| Victor | Repository setup and D2 AI usage section | 2–13 Nov |
| All seven | Rehearsal and D2 progress meeting | 16-17th November |
| Lee | Upload D2 | Deadline 20th November, uploaded ideally before between 16-19 |

## Database and API foundation

Show a bar from **16 to 20 November** for each of these tasks.

| Owner | Task |
| --- | --- |
| Ayaan | User database model, registration, login and shared permission checks |
| Lakhan | Game and genre models and game details |
| Haarisah | Platform and catalogue history models |
| Lee | Collection entries model |
| Zeeshan | Reviews model |
| Husray | Friendship and follow models |
| Victor | App setup, merging the database models, loading sample data and setting up Bruno |

## Parallel feature development and testing

Highlight **weeks 10–12** from 23 November to 13 December. Within each week, the individual bars run from Monday through Friday.

| Owner | Feature | 23–27 Nov | 30 Nov–4 Dec | 7–11 Dec |
| --- | --- | --- | --- | --- |
| Ayaan | Profiles and favourites | Build profiles and favourites | Test and help other members | Integrate and fix issues |
| Lakhan | Catalogue browsing and user content | Build lists and filters | Add guides and media links | Integrate and test |
| Haarisah | Catalogue maintenance | Build edits and change history | Build platform links | Integrate and test |
| Lee | Personal collections | Add and update entries | List, filter and remove entries | Integrate and test |
| Zeeshan | Reviews, ratings and shared test setup | Write and edit reviews, prepare shared test fixtures | Add ratings and review deletion | Integrate and test |
| Husray | Friends, follows and activity feed | Build friend requests and follows | Build the feed query | Run the full test suite |
| Victor | Forums, integration and Bruno collection | Build forum threads and posts | Add post rules and failure cases | Merge features and tidy the Bruno collection |

Add a separate bar for **all seven members from 23 November to 11 December** for Bruno API requests and automated Python tests. Show that success and failure tests are written alongside each feature.

## D3 demo, recording and submission

| Owner | 14–18 Dec | Follow-up work |
| --- | --- | --- |
| Ayaan | Prepare the introduction and record his segment | Slide deck, 21 Dec–3 Jan |
| Lakhan | Run the clean-machine test and record his segment | Edit and check the video, 21–31 Dec |
| Haarisah | Prepare the design comparison and record her segment | |
| Lee | Record the collections demo | Check that the video link works, 21–31 Dec. Submit the video link and ZIP during 4–10 Jan. |
| Zeeshan | Rehearse the demo and record his segment | |
| Husray | Prepare the evaluation script and record his segment | |
| Victor | Prepare the README and ZIP and record his segment | Final ZIP and AI usage log, 21 Dec–3 Jan |

December dates are in 2026 and January dates are in 2027. The January bar shows a planning window. Confirm the exact D3 deadline on Moodle.

## Checkpoints

| Checkpoint | Date | What the group checks |
| --- | --- | --- |
| CP0 | 3 Nov | Agree design decisions and workload. |
| CP1 | 13 Nov | Check and sign off the D2 document. |
| CP2 | 17 Nov | All seven attend the D2 meeting. Lee uploads the document before 15:00, subject to Moodle confirmation. |
| CP3 | 27 Nov | Set up a fresh copy, start the API and run the registration, login and browsing requests in Bruno. |
| CP4 | 11 Dec | Freeze features and run the full Bruno and automated Python test suites. |
| CP5 | 18 Dec | Complete the clean-machine test and record the code segments. |
| CP6 | 31 Dec | Check the finished video and confirm that someone outside the team can open and download it. |
| CP7 | Week of 4 Jan | Submit the D3 video link and ZIP. The plan lists 15:00 and the exact day is to be confirmed on Moodle. |

## Dependencies to show

- Draw gold arrows from **Ayaan’s shared login and permission checks** and **Victor’s merged models, sample data and Bruno setup**, due 20 November, into all seven feature rows starting the week of 23 November.
- Draw a gold arrow from the end of the group integration and testing bar into demo and recording preparation. Merged features at CP4 are also needed for the clean-machine test.
- Draw a gold arrow from **Lee’s video link check** to **final submission**.
- In the notes below the chart, explain that Lakhan’s game details use Haarisah’s platform links and Zeeshan’s ratings, Husray’s feed uses Lee’s collections and Zeeshan’s reviews, and Zeeshan’s shared test fixtures support the team’s automated tests. Use the sample data while the linked features are being built.

## If work slips

Raise work that is more than a week behind its checkpoint at the stand-up and pair the owner with a helper. Use a fixed test user if the shared login checks are late. If the feed is late, deliver friends and follows, demonstrate the feed with sample data and record the gap in the evaluation.

Drop Could items first, then late Should items. Keep Must features and Bruno failure tests. Record any change in the design change log. After CP4, limit work to bug fixes.

## How to recreate the image

- Use a wide, white landscape canvas. **3,600 by 2,646 pixels**. Put **CMP5387**, **Project Gantt chart** and the project name at the top left. Put the date range and **Planned schedule** at the top right.
- Put the task and owner columns on the left. Put ten equal weekly columns across the right, starting **2 Nov, 9 Nov, 16 Nov, 23 Nov, 30 Nov, 7 Dec, 14 Dec, 21 Dec, 28 Dec and 4 Jan**. Use white week labels on a dark forest-green header.
- Use four pale sage section bands in the order above. Add thin, light green gridlines. Shade the background of weeks 10–12 with a light green tint so the overlapping work is easy to see.
- Draw slim horizontal bars with rounded ends. Use deep green for design and setup, medium green for feature work, muted green for integration and tests, dark forest green for the meeting and upload, and light sage for final delivery work. Use gold for dependency arrows. Use dark text on pale bars and white text on darker bars.
- Use Helvetica or a similar clear sans-serif font. Make the task and owner labels readable at landscape page width. Keep text inside narrow bars short. Lee’s first two feature bars can be labelled **PUT** and **GET / DELETE**.
- Put checkpoints on a separate line below the bars. Use small green squares for CP0–CP6 and a light green span across 4–10 January for CP7. Put the checkpoint descriptions, dependency notes, and what to do if work slips beneath that line. Mark the D2 and D3 dates as subject to Moodle confirmation.
- When the plan changes, update the relevant row, checkpoint or dependency above. Use this brief to redraw the chart, export it as a high-resolution PNG, and check it at landscape page width before replacing the image in Word.

Sources: [Group plan, sections 8 and 9](CMP5387_Group_Plan.md), [team checklist](TEAM_TODO.md) and [progress tracker](progress%20tracker.md).
