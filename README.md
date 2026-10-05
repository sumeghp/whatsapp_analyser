# WhatsApp Chat Analyser

Turn an exported WhatsApp chat into charts, network diagrams, and stats: who talks the most, who replies to whom, when the chat is busiest, how activity changes over time, and the most-used words and emoji.

**Live app:** https://sumeghhp.gitlab.io/whatsapp_plots/

Everything runs in your browser. The chat file is never uploaded to a server.

---

## Contents

- [Features](#features)
- [Privacy](#privacy)
- [How to use](#how-to-use)
- [Exporting a chat from WhatsApp](#exporting-a-chat-from-whatsapp)
- [What each section shows](#what-each-section-shows)
- [Exporting results (PDF and CSV)](#exporting-results-pdf-and-csv)
- [Supported export formats](#supported-export-formats)
- [Stopwords](#stopwords)
- [Running locally](#running-locally)
- [Deploying on GitLab Pages](#deploying-on-gitlab-pages)
- [Project structure](#project-structure)
- [How the parser works](#how-the-parser-works)
- [Known limitations](#known-limitations)
- [Contributing](#contributing)
- [License](#license)

---

## Features

- **Summary strip:** total messages, participants, days covered, busiest day, peak hour, and longest daily streak.
- **Participants:** messages, words, words per message, media sent, and conversations started for each person.
- **Activity over time:** daily or weekly message counts, with quiet days shown as zero and the peak labelled.
- **Weekly heatmap:** messages by day of the week and hour of the day.
- **Conversational network graph:** interactive directed SVG network diagram showing who responds to whom, complete with message counts on nodes and reply volumes on links.
- **Top words:** for the whole chat or for one person, with common filler words removed.
- **Top emoji** used in the chat.
- **Export** a PDF report or CSV tables (including adjacency matrices and edge lists for Gephi/NetworkX).
- Reads both **Android and iPhone** exports, in **12-hour or 24-hour** time.
- Accepts the **`.txt`** file or the **`.zip`** WhatsApp creates.
- Works on phones and follows the system **light/dark** theme.
- A single HTML file with no build step and no external framework.

## Privacy

The file is read with the browser's `FileReader` API and analysed by JavaScript running on the page. No chat data is sent anywhere, and nothing is saved after you close the tab.

Exports (PDF and CSV) are also built in the browser and saved straight to your device.

The page loads only these from the internet:

| Resource | Purpose |
|---|---|
| Google Fonts | Typefaces. Without them the page falls back to system fonts. |
| [JSZip 3.10.1](https://cdnjs.com/libraries/jszip) from cdnjs | Opening `.zip` exports and creating the CSV `.zip`. `.txt` files work without it. |
| [html2canvas 1.4.1](https://cdnjs.com/libraries/html2canvas) and [jsPDF 2.5.1](https://cdnjs.com/libraries/jspdf) from cdnjs | Building the PDF report. Loaded only when you click **PDF report**. |

## How to use

1. Export a chat from WhatsApp (see below).
2. Open the app and drop the file onto the page, or click to choose it.
3. Use the header's **Choose an export** button to load a different chat.

## Exporting a chat from WhatsApp

**Android:** open the chat → **⋮** → **More** → **Export chat** → **Without media**

**iPhone:** open the chat → tap the contact or group name at the top → **Export Chat** → **Without Media**

Choosing *Without media* keeps the file small. Media messages are still counted, because the export marks where each one was sent.

## What each section shows

| Section | Details |
|---|---|
| **Messages** | Every message from a person, including media and deleted messages. System lines (joins, leaves, subject changes, encryption notices) are excluded. |
| **Days covered** | Calendar days from the first message to the last, including quiet days. |
| **Busiest day** | The single day with the most messages. |
| **Peak hour** | The hour of the day with the most messages across the whole chat. |
| **Longest streak** | The most consecutive days with at least one message. |
| **Words** | Words in text messages only. Media and deleted messages count as zero words, and links are ignored. |
| **Words/msg** | Words divided by text messages, so media doesn't pull the average down. |
| **Media** | Photos, videos, audio, stickers, GIFs, and documents. |
| **Starts** | How many times a person sent the first message after at least 6 hours of silence, a rough measure of who starts conversations. |
| **Activity over time** | Messages per day, or per week starting Monday. Hover or touch the chart to see exact counts. |
| **Heatmap** | Each cell shows the number of messages in that day-and-hour slot, and stronger colour means more messages. Blank cells had none, and the last column is each day's total. Counts of 1,000 or more are shortened (for example `1.2k`). Times are as they appear in the export, which is the phone's local time. |
| **Conversational Network** | A radial directed graph showing conversational turn-taking. Nodes display total messages sent by each participant, curved arrows show the direction of replies (Replier → Recipient), and badges on each link display the volume of replies between pairs. Links are inferred when a participant sends a message within 5 minutes of another or directly uses an `@mention`. |

## Exporting results (PDF and CSV)

Once a chat is loaded, click **Export** above the summary.

### PDF report

A report on A4 pages with every section: summary, participants, activity chart, heatmap, top words, top emoji, and the interactive network graph. It always uses the light theme, whatever theme the page is in, and every page is numbered.

The file is named after the chat, for example `college-friends-report.pdf` for *WhatsApp Chat with College Friends.txt*.

### CSV tables

Each table can be downloaded on its own, or all of them together as one `.zip`:

| File | Columns / Structure |
|---|---|
| `…-summary.csv` | Metric, Value: every number in the summary strip plus totals for words, media, deleted messages, and emoji |
| `…-participants.csv` | Name, Messages, Share of messages (%), Words, Words per message, Media, Deleted, Conversation starts |
| `…-daily-activity.csv` | Date, Weekday, Messages: one row per day, including quiet days |
| `…-day-hour-heatmap.csv` | Day, `00:00` … `23:00`, Total: the heatmap as a 7 × 24 grid |
| `…-reply-matrix-nxn.csv` | Replier \ Target, participant columns, Total Sent Replies: full $N \times N$ adjacency matrix |
| `…-reply-edge-list.csv` | Source, Target, Weight, Type: directed edge list ready for Gephi, Cytoscape, or NetworkX |
| `…-top-words.csv` | Person, Rank, Word, Count: top 50 for everyone, then top 25 for each person |
| `…-top-emoji.csv` | Rank, Emoji, Count: every emoji used |
| `…-messages.csv` | Date, Time, Author, Type, Message: the whole chat as a table |

About the CSV files:

- Dates are written as `YYYY-MM-DD` and times as 24-hour `HH:MM`, so spreadsheets sort them correctly.
- Files are UTF-8 with a byte-order mark, so Excel shows emoji, Hindi, and other scripts correctly.
- Messages that start with `=`, `+`, `-`, or `@` get a leading `'`, so a spreadsheet can't run them as formulas.
- Multi-line messages stay in one cell.

## Supported export formats

The parser recognises both line styles WhatsApp uses:

```text
Android:  05/10/26, 9:41 pm - Name: message
iPhone:   [05/10/26, 9:41:07 PM] Name: message