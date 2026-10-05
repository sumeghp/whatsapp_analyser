# WhatsApp Chat Analyser

Turn an exported WhatsApp chat into charts and stats: who talks the most, when the chat is busiest, how activity changes over time, and the most-used words and emoji.

**Live app:** https://sumeghhp.gitlab.io/whatsapp_plots/

Everything runs in your browser. The chat file is never uploaded to a server.

---

## Contents

- [Features](#features)
- [Privacy](#privacy)
- [How to use](#how-to-use)
- [Exporting a chat from WhatsApp](#exporting-a-chat-from-whatsapp)
- [What each section shows](#what-each-section-shows)
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

- **Summary strip:** total messages, participants, days covered, busiest day, peak hour and longest daily streak
- **Participants:** messages, words, words per message, media sent and conversations started, for each person
- **Activity over time:** daily or weekly message counts, with quiet days shown as zero and the peak labelled
- **Weekly heatmap:** messages by day of the week and hour of the day
- **Top words:** for the whole chat or for one person, with common filler words removed
- **Top emoji** used in the chat
- Reads both **Android and iPhone** exports, in **12-hour or 24-hour** time
- Accepts the **`.txt`** file or the **`.zip`** WhatsApp creates
- Works on phones and follows the system **light/dark** theme
- A single HTML file with no build step and no framework

## Privacy

The file is read with the browser's `FileReader` API and analysed by JavaScript running on the page. No chat data is sent anywhere, and nothing is saved after you close the tab.

The page loads only two things from the internet:

| Resource | Purpose |
|---|---|
| Google Fonts | Typefaces. Without them the page falls back to system fonts. |
| [JSZip 3.10.1](https://cdnjs.com/libraries/jszip) from cdnjs | Opening `.zip` exports. `.txt` files work without it. |

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
| **Media** | Photos, videos, audio, stickers, GIFs and documents. |
| **Starts** | How many times a person sent the first message after at least 6 hours of silence, a rough measure of who starts conversations. |
| **Activity over time** | Messages per day, or per week starting Monday. Hover or touch the chart to see exact counts. |
| **Heatmap** | Darker cells mean more messages in that day-and-hour slot. Times are as they appear in the export, which is the phone's local time. |

## Supported export formats

The parser recognises both line styles WhatsApp uses:

```text
Android:  05/10/26, 9:41 pm - Name: message
iPhone:   [05/10/26, 9:41:07 PM] Name: message
```

It also handles:

- 12-hour (`am`/`pm`, `AM`/`PM`) and 24-hour times
- The narrow no-break space newer WhatsApp versions put before `AM`/`PM`
- Day-first (`DD/MM/YY`) and month-first (`MM/DD/YY`) dates, detected from the data. If every date is ambiguous (both numbers 12 or under), day-first is assumed.
- `/`, `.` or `-` as date separators, and 2- or 4-digit years
- Messages that span several lines
- `<Media omitted>`, `image omitted`, `<attached: …>` and similar media markers
- "This message was deleted", "You deleted this message" and the `<This message was edited>` tag

## Stopwords

Stopwords are very common words that are left out of **Top words**, so the list shows words that say something about the chat. Without them the top results would always be "the", "you", "and", "hai" and "kya".

The list has 184 words in two groups.

**English (118):** articles, pronouns, helper verbs and chat fillers. For example: `the, and, you, that, this, have, are, was, but, not, what, just, yeah, okay, haha, lol, hmm, really, good, nice, today, tomorrow`

**Hindi / Hinglish (66):** common words typed in Roman script. For example: `hai, hain, nahi, nhi, kya, bhi, aur, haan, mein, tha, kar, raha, abhi, bas, kuch, sab, yaar, acha, theek, matlab, kyu, kaise, phir`

Some other words are skipped too, whether or not they're in the list:

- words shorter than 3 letters
- plain numbers
- links (`http://…`, `https://…`)
- punctuation and emoji (emoji are counted separately)

Only **Top words** uses this list. Message and word counts include every word.

### Editing the list

The list is the `STOP` set near the top of the script in `index.html`:

```js
const STOP = new Set(("the and for you that this with ... " +
  "toh hai hain nahi ...").split(" "));
```

Add or remove space-separated words inside the quotes. Words must be lowercase, because messages are lowercased before matching.

## Running locally

Open `index.html` in a browser. Everything except `.zip` support works straight from the file.

To serve it like the live site:

```bash
python3 -m http.server 8000
# then open http://localhost:8000
```

## Deploying on GitLab Pages

The app is one static file, so any Pages setup that publishes `index.html` will work. A minimal `.gitlab-ci.yml`:

```yaml
pages:
  stage: deploy
  script:
    - mkdir -p public
    - cp index.html public/
  artifacts:
    paths:
      - public
  rules:
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH
```

## Project structure

```text
.
├── index.html   # the whole app: markup, styles and script
├── README.md
└── LICENSE
```

Inside `index.html`, the script is split into four sections:

| Section | Responsibility |
|---|---|
| **Parsing** | `parseChat()` turns the raw export text into message objects `{ t, author, text, kind }` |
| **Analysis** | `analyse()` computes per-person stats, the daily series, heatmap, words, emoji and streaks |
| **Rendering** | `render*()` functions draw the summary, participants list, SVG chart, heatmap and rankings |
| **Input** | File picker, drag-and-drop, `.zip` handling and error messages |

## How the parser works

1. Invisible direction marks are removed from each line and special spaces are turned into normal spaces.
2. A line that matches the Android or iPhone timestamp pattern starts a new message. Any other non-empty line is added to the previous message, which is how multi-line messages are kept together.
3. Once every line is read, the date order is decided. A first number over 12 means day-first; a second number over 12 means month-first.
4. The time is converted to 24-hour, with 12 AM as 00:00 and 12 PM as 12:00.
5. The author is the text before the first `: `. Lines without an author, or whose "author" looks like a system event ("X changed the subject…", "X added Y"), are dropped.
6. Each message is tagged `text`, `media` or `deleted`.

## Known limitations

- Media, deleted and system-message detection uses **English** wording. Exports from a phone set to another language will still parse, but media and deleted messages may be counted as text.
- If someone changes their name or number during the chat, they appear as two people.
- Times come from the phone that exported the chat, with no time-zone conversion.
- Very large chats (hundreds of thousands of messages) may take a few seconds to analyse.

## Contributing

Issues and merge requests are welcome. Useful additions:

- Media and system-message wording for other languages
- More stopwords for other languages
- Sample exports from different phones and locales for testing the parser

## License

[MIT](LICENSE)
