<div align="center">

# ✅ Daily Tasks

**A zero-dependency daily task dashboard in a single HTML file.**

![HTML5](https://img.shields.io/badge/HTML5-E34F26?style=flat-square&logo=html5&logoColor=white) ![CSS3](https://img.shields.io/badge/CSS3-1572B6?style=flat-square&logo=css3&logoColor=white) ![JavaScript](https://img.shields.io/badge/Vanilla_JS-F7DF1E?style=flat-square&logo=javascript&logoColor=black) ![No dependencies](https://img.shields.io/badge/dependencies-0-brightgreen?style=flat-square) ![License: MIT](https://img.shields.io/badge/License-MIT-green?style=flat-square)

</div>

Open it each morning to see and manage the day's work. No server, no build step, no account — just one `index.html` file. Everything is saved in your browser.

## 🚀 Features

- **Priority dashboard** — tasks grouped into High / Medium / Low, each with a remaining-count badge
- **Live progress bar** — "4 of 9 done" updates on every check-off
- **Recurring vs one-off tasks** — recurring tasks reset automatically each new day; one-off tasks carry over until you delete them
- **Free-form categories** with suggestions (Work, Personal, Health, plus any you've used)
- **Clear completed** — bulk-remove finished one-off tasks
- **Accessible** — keyboard-operable checkboxes, ARIA labels and a live region for screen readers
- **Private by design** — data stays in `localStorage` on your device

## 🏁 Getting started

```bash
git clone https://github.com/DrPraveen-BA/-daily-tasks-landing.git
```

Then open `index.html` in any browser. That's it.

## 🗂️ How it works

Each task is stored as JSON under the `daily-tasks` key in `localStorage`. On every page load, recurring tasks whose `completedOn` date isn't today are unchecked — so the daily reset needs no background job. The full design is in [`docs/superpowers/specs`](docs/superpowers/specs/2026-06-05-daily-tasks-design.md).

## 🤝 Contributing

Ideas welcome! A few that are deliberately out of scope today and would make great first contributions: due dates, drag-and-drop reordering, and import/export. Open an [issue](../../issues) or a pull request.

## 📄 License

[MIT](LICENSE) © 2026 Dr Praveen GVS
