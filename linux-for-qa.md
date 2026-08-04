📘 Linux for QA Engineers: A Beginner-Friendly Guide to the Command Line

Author: Tatsiana Chakhovich Role: QA Automation Engineer

📌 Overview

Most of the applications we test don't run on our laptops — they run on Linux servers, inside Linux-based containers, out in a cloud environment somewhere. When something goes wrong in that environment, the fastest way to understand what happened is to be able to look around for yourself: check whether a file exists, see whether a process is still running, or read the last few lines of a log. You don't need to become a system administrator to do this. You just need to be comfortable with a small set of commands.

This guide walks through those commands one step at a time, starting from the very basics. Every example is something you can type yourself and see the result of immediately.

🧪 Where to Practice

The easiest way to follow along is on an actual Linux computer, so the commands behave exactly as shown. If you're on a Mac or Windows machine, the terminal looks similar but isn't quite the same system underneath, so a few commands may act differently. The simplest fix, if you have Docker installed, is to type:

bash
docker run -it ubuntu bash

That opens a small, temporary Linux environment inside a window, ready to try any command in this guide. If that sounds like too much for now, that's fine too — read through the guide first, and come back to try things hands-on once a Linux server or test environment is available to you at work, which is where you'll actually use these skills day to day.

🧠 1. Why Linux Matters for QA Engineers

A QA engineer who is comfortable at the command line can:

Check whether a file, config, or log actually exists, instead of guessing
Confirm whether a process or service is really running before blaming the test
Read a log directly instead of waiting on someone else to check it
Verify an API response with a single command before writing a full automated test
Reproduce an issue in the same environment it actually happened in

None of this requires becoming a Linux expert — just a working comfort with a small, consistent set of commands, covered step by step below.

⌨️ 2. Getting Comfortable in the Terminal

The terminal is just a place where you type commands instead of clicking things. Each command below follows the same shape: the command name, then optional flags (usually starting with a dash) that change how it behaves, then an argument such as a file or folder name that tells it what to act on.

bash
pwd
ls
ls -l
ls -la
cd my-folder
cd ..
cd ~
cat notes.txt
pwd — short for "print working directory." Takes no arguments; it just prints the full path of the folder you're currently in.
ls — lists the contents of a folder. With no argument, it lists the current folder; ls my-folder lists that folder instead.
ls -l — the -l flag means "long format": instead of just names, you get one line per file showing its permissions, owner, size, and last-modified date. You'll see this format again in the permissions section below.
ls -la — combines two flags: -l for long format plus -a, which means "all," including hidden files. On Linux, any file or folder whose name starts with a dot (like .bashrc or .git) is hidden from a plain ls, so -a is what reveals them. You can combine single-letter flags like this instead of writing ls -l -a.
cd my-folder — "change directory." The argument (my-folder) is the folder to move into; it can be a name in your current folder or a full path like cd /opt/app.
cd .. — .. always means "the folder one level up" from where you are.
cd ~ — the ~ character is a shortcut that always means your home folder, no matter where you currently are. cd by itself (no argument at all) does the same thing as cd ~.
cat notes.txt — "concatenate," but in everyday use it just means "print this file's contents to the screen." The argument is the file to display; you can list more than one file and cat will print them one after another.

A quick habit that helps a lot when you're starting out: run pwd and ls often, so you always know where you are and what's around you before running your next command.

🔐 3. Understanding File Permissions

Every file on Linux has an owner and a set of permissions that say who is allowed to read it, write to it, or run it. This matters for QA work because a surprising number of bugs turn out to be permission problems in disguise — a test that can't save a file, or a script that won't run.

bash
ls -l script.sh
# example output: -rwxr-xr-- 1 alice qa 812 script.sh

Reading that output from left to right: the first character (-) tells you it's a regular file (a d there would mean it's a folder instead). The next three letters (rwx) are what the owner of the file — shown a bit further along as alice — can do: read, write, and execute. The next three (r-x) are what the group — here, qa — can do. The last three (r--) are what everyone else on the system can do. The number after that (1) is a link count you can generally ignore, and 812 is the file's size in bytes.

If a script won't run and you see a permission like rw-r--r--, that's the clue: there's no x anywhere in it, so nobody is allowed to execute it yet. The fix uses chmod ("change mode"):

bash
chmod +x script.sh
chmod — the command that changes permissions. It always takes two things: what to change, and which file.
+x — the "what to change" part. + means add a permission, - would mean remove one, and x is the execute permission specifically (you could also use r or w for read or write). So +x reads as "add execute permission."
script.sh — the argument at the end: the file the change applies to.
📝 4. Viewing and Editing Files: more, less, and vi
more and less: reading a file without opening an editor

cat works fine for a short file, but for anything longer than a screenful, it just floods your terminal. more and less both show a file one screen at a time instead — less is the more capable of the two and is the one worth reaching for by default:

bash
more app.log
less app.log
more app.log — opens the file one page at a time. Press the space bar to move forward a page, Enter to move forward one line, and q to quit. It can only move forward.
less app.log — does the same job, but lets you move backward too (press b to go back a page), search inside the file (type /error and press Enter to jump to the next line containing "error"; press n to jump to the next match after that), and quit the same way, with q.

A simple rule of thumb: use cat for a quick peek at something small, and less for anything you expect to scroll through or search.

vi: a text editor that's (almost) always installed

Sometimes you need to change a file, not just look at it — for example, editing a config file on a test server that doesn't have a graphical text editor available. vi (or its more common modern version, vim) is worth knowing a handful of commands for, because it's installed on nearly every Linux system by default, even minimal ones.

bash
vi config.yaml

vi has two modes, and the biggest source of confusion for newcomers is not knowing which one they're in. When you open a file, you start in normal mode, where keys are commands rather than letters you're typing:

i — switches to insert mode, where you can type normally. Look for -- INSERT -- at the bottom of the screen to confirm you're in it.
Esc — exits insert mode and goes back to normal mode. If you're ever unsure which mode you're in, pressing Esc is always a safe move.
:w — (typed in normal mode, starting with a colon) saves the file without closing it.
:q — closes the file. vi will refuse if you have unsaved changes.
:wq — saves and closes in one step; probably the single most useful vi command to memorize.
:q! — closes without saving, discarding any changes. The ! is vi's way of saying "do it anyway, I know there are unsaved changes."

A four-command survival kit — i to start typing, Esc to stop, then :wq to save and quit — covers the vast majority of quick edits you'll ever need to make.

🔍 5. Finding Files and Searching Inside Them

Two commands cover almost everything you'll need here: find, for locating files, and grep, for searching the text inside them.

bash
find . -name "*.log"
find / -name "config.yaml" 2>/dev/null
find . -name "*.log" — the first argument (.) is where to start looking; . means "the current folder (and everything inside it)." -name is a flag that says "match by filename," and "*.log" is the pattern to match — the * is a wildcard meaning "anything," so this matches any file ending in .log. Quoting the pattern keeps the terminal from trying to interpret the * itself.
find / -name "config.yaml" 2>/dev/null — same command, but starting from / (the very top of the filesystem), so it searches everywhere. The 2>/dev/null at the end isn't part of find itself — it's a general terminal instruction that throws away error messages (like "Permission denied" from folders you can't peek into) so only real matches show up.

find works everywhere and always searches what's actually on disk right now, which makes it a safe default to reach for. (You may also see people use a command called locate — it can be faster, but it depends on a separate tool that isn't installed on every system, so find is the more dependable one to learn first.)

bash
grep "error" app.log
grep -i "error" app.log
grep -c "error" app.log
grep -n "error" app.log
grep "error" app.log — the first argument is the text pattern to search for, the second is the file to search in. It prints every line from app.log that contains the word "error."
-i — makes the search case-insensitive, so it also matches "Error" or "ERROR."
-c — instead of printing matching lines, prints just a count of how many there are.
-n — prints the line number next to each match, which is handy when you want to open the file and jump straight to that spot.

grep is one of the most useful tools for QA work: instead of opening a huge log file and scrolling, you ask it directly for the lines you care about.

⚙️ 6. Checking What's Running

Sometimes a test fails because the application, or a service it depends on, isn't actually running. This command shows you every running process:

bash
ps aux
ps aux | grep java
ps aux — ps lists processes; aux is a set of options written together (not three separate arguments): a shows processes for all users, u displays them in a more readable, detailed format, and x includes background processes that aren't tied to a terminal window. Together, aux is the combination people use almost every time.
| grep java — the | (called a "pipe") takes the output of ps aux and feeds it into grep as if it were a file, so you only see the lines mentioning "java" instead of the entire process list.

If something needs to be stopped — for example, a test server that got stuck — you first find its process number (called a PID) in that list, then stop it:

bash
kill 12345
kill -9 12345
kill 12345 — the argument is the PID (process ID) you want to stop, found from the ps aux output. By default this sends a polite request (technically called SIGTERM, signal 15) asking the process to shut down cleanly.
kill -9 12345 — the -9 flag sends signal number 9 (SIGKILL) instead, which stops the process immediately without giving it a chance to clean up. Use the plain kill first, and only reach for -9 if the process doesn't respond.
🪵 7. Reading Logs When Something Breaks

When a test fails and you're not sure why, the log file is usually where the answer is. A few simple ways to look at one:

bash
cat app.log
tail app.log
tail -f app.log
grep -i "error" app.log | tail -20
cat app.log — prints the entire file at once; fine for small files, but overwhelming for anything large.
tail app.log — by default prints just the last 10 lines of the file, which is usually the most relevant part when something just failed.
tail -f app.log — the -f flag means "follow": instead of stopping after printing, it keeps the terminal open and prints new lines as they get written to the file.
tail -n 50 app.log — worth knowing as a variation: -n 50 changes how many lines are shown, from the default of 10 up to 50.
grep -i "error" app.log | tail -20 — combines two commands with a pipe: grep -i "error" app.log finds every error line (case-insensitive), and | tail -20 then narrows that down to just the most recent 20 of those matches.

tail -f is especially useful while a test is running: open it in one window, then run your test in another, and watch what gets written in real time.

🌐 8. Testing an API from the Terminal

Before writing an automated test, it's often quicker to just ask the API directly and see what comes back. The curl command does exactly that:

bash
curl https://api.example.com/health
curl -i https://api.example.com/health
curl -X POST https://api.example.com/users \
  -H "Content-Type: application/json" \
  -d '{"name":"Test User"}'
curl https://api.example.com/health — the argument is the URL to request. With no flags, curl sends a GET request (the same kind your browser sends to load a page) and prints the response body.
-i — includes the response status code and headers above the body, so you can see things like whether it returned 200 (success) or 404 (not found), not just the content.
-X POST — tells curl which HTTP method to use instead of the default GET. POST is the method typically used to create or submit something.
-H "Content-Type: application/json" — adds a header to the request; this particular one tells the server the data you're sending is JSON. You can repeat -H to add more than one header.
-d '{"name":"Test User"}' — the -d flag provides the request body ("data") to send along, here a small piece of JSON.

This is a fast way to confirm whether a problem is in the API itself or somewhere else, before you spend time writing a full test around it.

🐳 9. A Gentle Introduction to Docker

You'll often hear Docker mentioned in QA work because it lets a team package an application, along with everything it needs to run, into one consistent unit called a container — so it behaves the same on every machine, including yours. You don't need to build these containers yourself to get value from Docker; a few commands for looking at ones that are already running go a long way:

bash
docker ps
docker logs my-container
docker exec -it my-container bash
docker ps — lists containers that are currently running, along with a name or ID for each one, which you'll need for the next two commands.
docker logs my-container — prints the log output from the container named my-container. Add -f the same way you would with tail -f (docker logs -f my-container) to keep watching new log lines as they come in.
docker exec -it my-container bash — docker exec runs a command inside a container that's already running. -i and -t are two flags almost always used together: -i ("interactive") keeps the connection open so you can type, and -t ("tty") makes it display like a normal terminal window rather than raw text. my-container is the argument saying which container to run this in, and bash is the actual command being run — in this case, starting a shell so you can keep typing more commands once you're inside.

That last command is worth remembering: it drops you inside the container itself, where you can then use everything from the earlier sections — ls, cat, tail, grep — exactly as if you'd logged into a small Linux computer.

🔧 10. Shell Scripting: Automating the Repetitive

Once a handful of commands are comfortable on their own, the next natural step is putting them together into a script — a text file full of commands that runs from top to bottom, so you don't have to type the same sequence by hand every time. For a QA engineer, this is often the highest-value skill in this whole guide: a five-line script can turn a five-minute manual checklist into something that runs the same way, every time, in one step.

A first script
bash
#!/usr/bin/env bash

echo "Starting checks..."
today=$(date +%Y-%m-%d)
echo "Today is $today"
#!/usr/bin/env bash — called the "shebang" line, always the first line of a script. It tells the system which program should run the rest of the file — here, bash, the most common shell. You'll always include this line, but you won't need to change it.
echo "Starting checks..." — echo simply prints its argument (the text in quotes) to the screen. It's the easiest way for a script to tell you what it's doing as it runs.
today=$(date +%Y-%m-%d) — creates a variable named today and stores a value in it. The $(...) part runs the command inside it (date +%Y-%m-%d, which prints today's date) and captures its output instead of printing it directly.
echo "Today is $today" — to use a variable's value later, put a $ in front of its name. Inside double quotes, $today is replaced with whatever was stored in it.

To run a script like this, save it as a file (for example check.sh), make it executable the same way you did earlier in this guide, and then run it:

bash
chmod +x check.sh
./check.sh
./check.sh — the ./ in front matters: it tells the shell "run the script sitting right here in the current folder," rather than searching elsewhere on the system for a program with that name.
Adding a condition and a loop

Two more building blocks cover most of what comes up in everyday QA scripts: checking a condition with if, and repeating an action with a loop.

bash
if [ "$today" = "2026-08-04" ]; then
  echo "It's release day."
else
  echo "Just a regular day."
fi
if [ condition ]; then ... else ... fi — reads almost like plain English: if the condition in brackets is true, run the first block; otherwise run the else block. fi ("if" spelled backwards) marks the end of the block.
[ "$today" = "2026-08-04" ] — the condition itself: the brackets are the test, and here it checks whether the today variable equals a specific date. The spaces right after [ and before ] are required, not optional.
bash
endpoints=("https://api.example.com/health" "https://api.example.com/status")

for url in "${endpoints[@]}"; do
  echo "Checking $url"
  curl -s -o /dev/null -w "%{http_code}\n" "$url"
done
endpoints=(...) — creates a list (called an array) of values, here two URLs, so the script can loop over them one at a time instead of repeating the same lines twice.
for url in "${endpoints[@]}"; do ... done — the loop itself: for each item in the endpoints list, store it in a variable called url and run everything between do and done once per item.
curl -s -o /dev/null -w "%{http_code}\n" "$url" — a quieter version of the curl command from earlier: -s hides curl's normal progress output, -o /dev/null throws away the response body (since here you only care whether it succeeded), and -w "%{http_code}\n" prints just the status code, like 200 or 404, instead.

Put together, that loop is a small but genuinely useful smoke test: run it before a full regression suite to get an instant answer to "is the environment even up?" instead of finding out twenty minutes into a test run.

A few habits specific to scripts
Add set -e near the top of a script (right after the shebang line) so it stops immediately if any command fails, instead of plowing ahead and producing confusing results further down.
Keep scripts in the same version control repository as the tests they support, so changes to one are reviewed alongside changes to the other.
Start small. A script that replaces one manual step you personally repeat often is worth more than an ambitious one you never finish.
✅ 11. Good Habits to Build From Here
Keep a small notes file of commands you've used and what they were for — you'll reuse most of them constantly.
When you're not sure what a command does, run man <command> (for example, man grep) to see its built-in manual, or search for it — there's no shame in looking things up every time at first.
Be careful with any command involving rm (delete) — there's no undo, so double-check the file or folder name before pressing enter.
Practice on a real Linux environment when you can (a work server, staging environment, or a Docker container) rather than guessing based on a Mac or Windows terminal, since small differences can be confusing early on.
🏁 Conclusion

None of this requires becoming a Linux expert overnight. Learning even this set of basics — moving around, checking permissions, viewing and editing files, searching files and logs, checking what's running, testing an API directly, and stringing commands together into a script — already covers most of what comes up day to day in QA work. The rest builds naturally, one real situation at a time, as you keep using it.

👤 About the Author

Tatsiana Chakhovich is a QA Engineer at CaringBridge, where she focuses on the reliability and quality of software that helps people stay connected during health journeys. She writes about test engineering, automation, and making technical skills approachable for engineers who are building their expertise.
