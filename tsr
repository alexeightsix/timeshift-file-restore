#!/usr/bin/env node
const fs = require('fs');
const { exec } = require("child_process");
const prompts = require('prompts');
const p = require('path');

exec("which timeshift", (err) => {
    if (err) {
        console.log("timeshift binary not found!");
        process.exit(1);
    }
});

function parseDate(string) {
    const regex = /(^[^d]{4})-([^d]{2})-([^d]{2})_([^d]{2})-([^d]{2})-([^d]{2}$)/gm;
    let m = regex.exec(string);
    return new Date(m[1] + "-" + m[2] + "-" + m[3] + "T" + m[4] + ":" + m[5] + ":" + m[6]);
}

function setPath(path) {
    if (path == undefined || path.length == 0)
        return null;

    return path.split("/").length == 1 ? process.cwd() + "/" + path : path
}

const path = setPath(process.argv[2]);

if (!path)
    process.exit(2);

exec("find /run/timeshift/*/*/snapshots/*/*/" + path + " | sort -n", (err, stdout) => {
    let files = stdout.split("\n").filter((line) => line !== '').map((path, index) => {
        let date = path.split("/")[6];
        date = parseDate(date).toUTCString()

        var trimmed = "/" + path.split("/").filter((seg, index) => index > 7).join("/")
        path = { index: index, date: date, path: path, pathTrimmed: trimmed }
        return path;
    })

    if (files.length == 0) {
        console.log("\x1b[31m", "🚫 " + path);
        process.exit(1);
    }

    files.forEach((file) => {
        console.log(file.date + "  [ " + (file.index) + " ]    " + file.pathTrimmed);
    });

    var file = null;

    (async () => {
        const response = await prompts({
            type: 'number',
            name: 'value',
            message: 'Which file do you wish to restore?',
            validate: (value) => {
                file = files.find((f, idx) => idx === value)
                return (file != null);
            }
        }).then(() => {
            const source = file.path;
            const dest = process.cwd() + "/" + p.basename(file.path);

            fs.copyFileSync(source, dest);
            console.log("✔" + source + " ➡️ " + dest);
        })
    })();
})