> Originally published at https://diegoferreyra.com/writing/a-backup-skill-your-ai-agent-installs

# A backup tool your AI agent sets up for you

I wanted one thing: my project folder copied to the cloud, everything except
the code (that already lives in git). An hour of back-and-forth with an AI
agent later, it had turned into a small open-source tool,
[dfd_sync](https://github.com/42pe/dfd_sync), that **an agent installs and
configures for you**.

You don't download it or read a manual. You paste one line into your agent:

```text
Revisa https://github.com/42pe/dfd_sync y ayúdame a configurar el skill de backups automáticos en Google Drive.
```

The agent reads the repo, installs the instructions, explains each step in
plain words, asks you what to back up, and leaves the first folder backing
itself up every few hours.

> **No warranty.** It's a personal, free tool. It does its best, but it can
> fail without telling you. It doesn't replace a professional backup. Check
> your Drive now and then.

## How it came together

The whole tool came out of one conversation with Claude Code. The short
version:

1. **Where to sync.** I had Google Drive and iCloud. The agent advised
   against both apps' own sync: moving the folder breaks paths, neither lets
   you skip a subfolder cleanly, and live sync corrupts git repos. Better: a
   one-way copy with [rclone](https://rclone.org), leaving the folder where
   it is.
2. **Settings live with the folder.** I asked for every setting to sit in a
   small file inside the folder, so the backup describes itself. Then a
   global file listing the parent folders to scan, so any folder can join
   later.
3. **One skill, three situations.** First time ever, a new folder, or a
   folder that's already set up. Each gets its own short conversation.
4. **History without surprises.** We argued about the size limit for old
   versions until the rule was simple: delete the oldest first, never the
   newest, and the copy itself always happens.
5. **Testing the experience, not just the code.** I asked for two separate
   sessions to walk through setup as a dry run, one from the skill and one
   from the README prompt. They found bugs and, more useful, the places where
   a non-technical person would get lost.
6. **Ship it.** Plain-language rewrite, a no-warranty notice up front, a
   clean history, public repo.

I didn't write the code. I made the decisions, and pushed back when a choice
didn't match how I wanted it to behave.

## The principles

- **One-way copy.** Your Mac → Drive. Nothing in Drive ever overwrites your
  files.
- **The folder describes its own backup.** Open the settings file and you
  know where it goes, how often, and what's skipped.
- **The agent explains, you decide.** It asks before anything that matters:
  connecting your Google account, deleting an old copy, backing up your whole
  home folder.
- **No jargon.** It assumes you're not technical, and says what each step is
  for before asking.
- **Recent history is sacred.** Old versions expire after 90 days or when
  they pass 1 GB, always oldest first.

## How it works, in plain words

**A tiny program does the copying.** It's called **rclone**: free, open
source, and good at one job, copying files to cloud storage. It only copies
what changed.

**It needs permission to enter your Google Drive.** Google doesn't let any
program in without you. So once, your browser opens, you sign in with your
Google account and click *Allow*. rclone keeps that permission (a "remote",
named `gdrive`) and renews it by itself. You can take it away anytime from
your Google account's security settings.

**An alarm wakes up every hour.** Macs have a built-in scheduler called
**cron**: a list of tasks with times. The agent adds one line to it: every
hour, on the hour, run the backup. Each folder then decides whether it's its
turn, every 12 hours by default.

**The settings are hidden files.** Files whose name starts with a dot are
hidden in Finder, so they don't clutter your folders. There are two:

- `~/.dfd_bkp_sync`, in your home folder: which areas of your Mac to look in,
  the history limits, and when the alarm last rang.
- `.dfd_bkp`, inside each backed-up folder: where it goes in Drive, how
  often, what to skip, and whether it's paused.

To see them, press **⌘ ⇧ .** in Finder. They're plain text; the agent edits
them for you.

**Old versions go to a dated folder.** Before replacing a file in Drive, the
old one moves to `Backups/_archive/<folder>/<date-time>/`. If you break
something, the previous version is there.

## Mac only, for now

It works on macOS today. The ideas are simple, though: a copy program, a
scheduler, a settings file per folder. On Windows the scheduler is *Task
Scheduler* instead of cron, and rclone runs there too. Ask your agent to
adapt it while keeping the same principles, and it will probably manage.

## Try it

It works with Claude Code, ChatGPT or any agent that can run commands on
your computer. The repo is at
[github.com/42pe/dfd_sync](https://github.com/42pe/dfd_sync). Paste the
prompt above and follow along.
