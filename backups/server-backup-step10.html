const express = require("express");
const path = require("path");
const fs = require("fs");
const { spawn } = require("child_process");

const app = express();
const PORT = 3000;

const ROOT = __dirname;
const DOWNLOAD_DIR = path.join(ROOT, "downloads");
const HISTORY_FILE = path.join(ROOT, "history.json");
const YTDLP = path.join(ROOT, "yt-dlp.exe");

app.use(express.json());
app.use(express.urlencoded({ extended: true }));

if (!fs.existsSync(DOWNLOAD_DIR)) {
    fs.mkdirSync(DOWNLOAD_DIR, { recursive: true });
}

if (!fs.existsSync(HISTORY_FILE)) {
    fs.writeFileSync(HISTORY_FILE, "[]", "utf8");
}

// ==============================
// STATE
// ==============================

let queue = [];
let processing = false;
let activeId = null;
let activeProcess = null;

let progress = {
    percent: 0,
    speed: "-",
    size: "-",
    eta: "-",
    filename: "",
    status: "idle"
};

// ==============================
// HELPERS
// ==============================

function id() {
    return Date.now().toString() + Math.random().toString(36).slice(2, 8);
}

function historyRead() {
    try {
        return JSON.parse(fs.readFileSync(HISTORY_FILE, "utf8"));
    } catch {
        return [];
    }
}

function historyWrite(data) {
    fs.writeFileSync(HISTORY_FILE, JSON.stringify(data, null, 2), "utf8");
}

function addHistory(job, filename) {
    const data = historyRead();

    data.unshift({
        id: id(),
        title: job.title || "Downloaded File",
        filename: filename || "",
        type: job.type,
        url: job.url,
        quality: job.quality || "",
        format: job.format || "",
        date: new Date().toISOString()
    });

    historyWrite(data);
}

function resetProgress() {
    progress = {
        percent: 0,
        speed: "-",
        size: "-",
        eta: "-",
        filename: "",
        status: "idle"
    };
}

function publicQueue() {
    return queue.map(x => ({
        id: x.id,
        url: x.url,
        title: x.title,
        type: x.type,
        quality: x.quality,
        format: x.format,
        status: x.status,
        filename: x.filename || "",
        error: x.error || "",
        date: x.date
    }));
}

function latestFile(before) {
    const files = fs.readdirSync(DOWNLOAD_DIR);

    const found = files
        .filter(x => !before.includes(x))
        .map(x => {
            const p = path.join(DOWNLOAD_DIR, x);

            try {
                return {
                    name: x,
                    time: fs.statSync(p).mtimeMs
                };
            } catch {
                return null;
            }
        })
        .filter(Boolean)
        .sort((a, b) => b.time - a.time);

    return found.length ? found[0].name : "";
}

// ==============================
// PROGRESS PARSER
// ==============================

function parseProgress(text) {
    const percent = text.match(/PERCENT=([0-9.]+)/i);
    const speed = text.match(/SPEED=(.*?)\s+SIZE=/i);
    const size = text.match(/SIZE=(.*?)\s+ETA=/i);
    const eta = text.match(/ETA=(.*?)\s+FILENAME=/i);
    const filename = text.match(/FILENAME=(.*)$/i);

    if (percent) {
        progress.percent = Math.min(100, Number(percent[1]) || 0);
    }

    if (speed) {
        progress.speed = speed[1].trim() || "-";
    }

    if (size) {
        progress.size = size[1].trim() || "-";
    }

    if (eta) {
        progress.eta = eta[1].trim() || "-";
    }

    if (filename) {
        progress.filename = filename[1].trim();
    }
}

// ==============================
// VIDEO ARGS
// ==============================

function videoArgs(job) {
    let format;

    if (job.quality === "best") {
        format = "bestvideo+bestaudio/best";
    } else {
        const h = Number(job.quality) || 720;
        format = `bestvideo[height<=${h}]+bestaudio/best[height<=${h}]/best`;
    }

    return [
        "--newline",
        "--no-playlist",
        "--progress",
        "--progress-template",
        "PERCENT=%(progress._percent_str)s SPEED=%(progress._speed_str)s SIZE=%(progress._total_bytes_str)s ETA=%(progress._eta_str)s FILENAME=%(progress.filename)s",
        "-f",
        format,
        "--merge-output-format",
        job.format === "webm" ? "webm" : "mp4",
        "-o",
        path.join(DOWNLOAD_DIR, "%(title)s.%(ext)s"),
        job.url
    ];
}

// ==============================
// MP3 ARGS
// ==============================

function mp3Args(job) {
    return [
        "--newline",
        "--no-playlist",
        "--progress",
        "--progress-template",
        "PERCENT=%(progress._percent_str)s SPEED=%(progress._speed_str)s SIZE=%(progress._total_bytes_str)s ETA=%(progress._eta_str)s FILENAME=%(progress.filename)s",
        "-x",
        "--audio-format",
        "mp3",
        "--audio-quality",
        job.quality || "192K",
        "-o",
        path.join(DOWNLOAD_DIR, "%(title)s.%(ext)s"),
        job.url
    ];
}

// ==============================
// RUN JOB
// ==============================

function runJob(job) {
    return new Promise((resolve, reject) => {
        resetProgress();

        const before = fs.readdirSync(DOWNLOAD_DIR);

        activeId = job.id;
        job.status = "downloading";

        progress.status = "downloading";

        const args = job.type === "mp3"
            ? mp3Args(job)
            : videoArgs(job);

        activeProcess = spawn(YTDLP, args, {
            cwd: ROOT,
            windowsHide: true
        });

        let errorText = "";

        activeProcess.stdout.on("data", data => {
            const text = data.toString();

            text.split(/\r?\n/).forEach(line => {
                if (line.trim()) {
                    parseProgress(line);
                }
            });
        });

        activeProcess.stderr.on("data", data => {
            errorText += data.toString();
        });

        activeProcess.on("error", error => {
            activeProcess = null;
            reject(error);
        });

        activeProcess.on("close", code => {
            activeProcess = null;

            if (code !== 0) {
                progress.status = "failed";

                reject(
                    new Error(
                        errorText.trim() ||
                        `Download failed. Exit code: ${code}`
                    )
                );

                return;
            }

            let filename = progress.filename;

            if (filename) {
                filename = path.basename(filename);
            }

            if (!filename || !fs.existsSync(path.join(DOWNLOAD_DIR, filename))) {
                filename = latestFile(before);
            }

            job.filename = filename;
            job.status = "completed";

            progress.percent = 100;
            progress.status = "completed";
            progress.filename = filename;
            progress.eta = "00:00";

            addHistory(job, filename);

            resolve(filename);
        });
    });
}

// ==============================
// QUEUE PROCESSOR
// ==============================

async function processQueue() {
    if (processing) return;

    processing = true;

    try {
        while (true) {
            const job = queue.find(x => x.status === "waiting");

            if (!job) break;

            try {
                await runJob(job);
            } catch (error) {
                job.status = "failed";
                job.error = error.message || "Download failed";
                progress.status = "failed";
            }

            activeId = null;
        }
    } finally {
        processing = false;
        activeId = null;
        activeProcess = null;
    }
}

// ==============================
// INFO API
// ==============================

app.post("/api/info", (req, res) => {
    const url = req.body && req.body.url;

    if (!url) {
        return res.status(400).json({
            success: false,
            error: "URL is required"
        });
    }

    const child = spawn(
        YTDLP,
        [
            "--dump-single-json",
            "--no-playlist",
            "--skip-download",
            url
        ],
        {
            cwd: ROOT,
            windowsHide: true
        }
    );

    let output = "";
    let errorOutput = "";

    child.stdout.on("data", data => {
        output += data.toString();
    });

    child.stderr.on("data", data => {
        errorOutput += data.toString();
    });

    child.on("error", error => {
        res.status(500).json({
            success: false,
            error: error.message
        });
    });

    child.on("close", code => {
        if (code !== 0) {
            return res.status(400).json({
                success: false,
                error: errorOutput.trim() || "Unable to fetch video information"
            });
        }

        try {
            const info = JSON.parse(output);

            const heights = [
                ...new Set(
                    (info.formats || [])
                        .map(x => Number(x.height))
                        .filter(x => x > 0)
                )
            ].sort((a, b) => a - b);

            res.json({
                success: true,
                title: info.title || "Unknown Title",
                thumbnail: info.thumbnail || "",
                duration: info.duration || 0,
                uploader: info.uploader || "",
                webpage_url: info.webpage_url || url,
                availableHeights: heights
            });
        } catch (error) {
            res.status(500).json({
                success: false,
                error: "Could not read video information"
            });
        }
    });
});

// ==============================
// DETECT
// ==============================

app.post("/api/detect", (req, res) => {
    const url = req.body && req.body.url;

    if (!url) {
        return res.json({
            success: false,
            type: "unknown"
        });
    }

    let type = "unknown";

    if (
        url.includes("youtube.com") ||
        url.includes("youtu.be")
    ) {
        type = "youtube";
    }

    res.json({
        success: true,
        type
    });
});

// ==============================
// DOWNLOAD VIDEO
// ==============================

app.post("/api/download", (req, res) => {
    const {
        url,
        quality = "best",
        format = "mp4",
        title = "Video"
    } = req.body || {};

    if (!url) {
        return res.status(400).json({
            success: false,
            error: "URL is required"
        });
    }

    const job = {
        id: id(),
        url,
        title,
        type: "video",
        quality,
        format,
        status: "waiting",
        filename: "",
        error: "",
        date: new Date().toISOString()
    };

    queue.push(job);
    processQueue();

    res.json({
        success: true,
        item: job
    });
});

// ==============================
// DOWNLOAD MP3
// ==============================

app.post("/api/mp3", (req, res) => {
    const {
        url,
        quality = "192K",
        title = "Audio"
    } = req.body || {};

    if (!url) {
        return res.status(400).json({
            success: false,
            error: "URL is required"
        });
    }

    const job = {
        id: id(),
        url,
        title,
        type: "mp3",
        quality,
        format: "mp3",
        status: "waiting",
        filename: "",
        error: "",
        date: new Date().toISOString()
    };

    queue.push(job);
    processQueue();

    res.json({
        success: true,
        item: job
    });
});

// ==============================
// QUEUE
// ==============================

app.get("/api/queue", (req, res) => {
    res.json({
        success: true,
        queue: publicQueue(),
        count: queue.length,
        processing,
        activeQueueItemId: activeId
    });
});

app.delete("/api/queue/:id", (req, res) => {
    const item = queue.find(x => x.id === req.params.id);

    if (!item) {
        return res.status(404).json({
            success: false,
            error: "Queue item not found"
        });
    }

    if (item.status === "downloading") {
        return res.status(400).json({
            success: false,
            error: "Active download cannot be removed"
        });
    }

    queue = queue.filter(x => x.id !== req.params.id);

    res.json({
        success: true
    });
});

app.post("/api/queue/:id/retry", (req, res) => {
    const item = queue.find(x => x.id === req.params.id);

    if (!item) {
        return res.status(404).json({
            success: false,
            error: "Queue item not found"
        });
    }

    item.status = "waiting";
    item.error = "";
    item.filename = "";

    processQueue();

    res.json({
        success: true
    });
});

app.delete("/api/queue", (req, res) => {
    queue = queue.filter(x => x.status === "downloading");

    res.json({
        success: true
    });
});

// ==============================
// PROGRESS
// ==============================

app.get("/api/progress", (req, res) => {
    res.json({
        success: true,
        ...progress,
        activeQueueItemId: activeId
    });
});

// ==============================
// CANCEL
// ==============================

app.post("/api/cancel", (req, res) => {
    if (!activeProcess) {
        return res.json({
            success: false,
            error: "No active download"
        });
    }

    try {
        activeProcess.kill();
    } catch {}

    const item = queue.find(x => x.id === activeId);

    if (item) {
        item.status = "failed";
        item.error = "Cancelled by user";
    }

    activeId = null;
    resetProgress();

    res.json({
        success: true
    });
});

// ==============================
// FILE DOWNLOAD
// ==============================

app.get("/api/download-file", (req, res) => {
    const filename = req.query.filename;

    if (!filename) {
        return res.status(400).send("Filename required");
    }

    const safe = path.basename(filename);
    const file = path.join(DOWNLOAD_DIR, safe);

    if (!fs.existsSync(file)) {
        return res.status(404).send("File not found");
    }

    res.download(file, safe);
});

// ==============================
// HISTORY
// ==============================

app.get("/api/history", (req, res) => {
    res.json({
        success: true,
        history: historyRead()
    });
});

app.delete("/api/history/:id", (req, res) => {
    const data = historyRead().filter(
        x => x.id !== req.params.id
    );

    historyWrite(data);

    res.json({
        success: true
    });
});

app.delete("/api/history", (req, res) => {
    historyWrite([]);

    res.json({
        success: true
    });
});

// ==============================
// WEBSITE FALLBACK
// IMPORTANT: Express 5 compatible
// ==============================

app.use((req, res) => {
    res.sendFile(path.join(ROOT, "index.html"));
});

// ==============================
// START
// ==============================

app.listen(PORT, () => {
    console.log("");
    console.log("====================================");
    console.log("        URLSniper Started");
    console.log("====================================");
    console.log(`URLSniper running at http://localhost:${PORT}`);
    console.log("Queue System: ENABLED");
    console.log("Advanced Downloader: ENABLED");
    console.log("");
});