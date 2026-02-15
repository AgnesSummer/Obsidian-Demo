```dataviewjs
const year = 2026;
const colorScheme = ['#E0E5E7', '#A8D5BA', '#F7E6A2', '#F0B87E', '#E8A0A0'];
const monthNames = ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec'];

// 获取文件修改日期
const files = dv.pages()
    .filter(p => p.file.mtime)
    .map(p => ({
        date: moment(p.file.mtime.toString()).format('YYYY-MM-DD'),
        count: 1
    }));

// 按日期聚合
const dateMap = {};
files.forEach(f => {
    dateMap[f.date] = (dateMap[f.date] || 0) + 1;
});

// 生成日历数据
const startDate = moment(`${year}-01-01`);
const endDate = moment(`${year}-12-31`);
let weeks = [];
let currentWeek = [];

// 填充年初的空白
for (let i = 0; i < startDate.day(); i++) {
    currentWeek.push(null);
}

const monthPositions = [];
for (let d = startDate.clone(); d.isSameOrBefore(endDate); d.add(1, 'day')) {
    const dateStr = d.format('YYYY-MM-DD');
    const count = dateMap[dateStr] || 0;

    // 记录每月第一天位置
    if (d.date() === 1) {
        monthPositions.push({ weekIndex: weeks.length, month: d.month() });
    }

    currentWeek.push({
        date: dateStr,
        count: count,
        level: Math.min(count > 0 ? 1 + Math.floor((count - 1) / 2) : 0, 4)
    });

    if (currentWeek.length === 7) {
        weeks.push(currentWeek);
        currentWeek = [];
    }
}

// 处理最后一周不足7天的情况
if (currentWeek.length > 0) {
    while (currentWeek.length < 7) {
        currentWeek.push(null);
    }
    weeks.push(currentWeek);
}

// 渲染热力图
const container = dv.container;
container.innerHTML = `
<style>
@import url('https://fonts.googleapis.com/css2?family=Figtree:wght@400;500;600&display=swap');
.heatmap-wrapper {
    font-family: 'Figtree', BlinkMacSystemFont, sans-serif;
    padding: 16px;
}
.heatmap-title {
    font-size: 0.9em;
    font-weight: 600;
    margin-bottom: 8px;
    color: #4384A3;
}
.heatmap-months {
    display: flex;
    margin-left: 46px;
    margin-bottom: 4px;
    height: 16px;
    position: relative;
}
.month-label {
    position: absolute;
    font-size: 11px;
    color: #4384A3;
    white-space: nowrap;
    transform: translateX(-50%);
}
.heatmap-body {
    display: flex;
    gap: 3px;
    align-items: flex-start;
}
.weekday-labels {
    display: flex;
    flex-direction: column;
    gap: 3px;
    font-size: 10px;
    color: #4384A3;
    line-height: 10px;
}
.weekday-label {
    height: 10px;
}
.heatmap {
    display: flex;
    gap: 3px;
}
.week {
    display: flex;
    flex-direction: column;
    gap: 3px;
}
.day {
    width: 10px;
    height: 10px;
    border-radius: 3px;
    background: ${colorScheme[0]};
    position: relative;
}
.day[data-level="1"] { background: ${colorScheme[1]}; }
.day[data-level="2"] { background: ${colorScheme[2]}; }
.day[data-level="3"] { background: ${colorScheme[3]}; }
.day[data-level="4"] { background: ${colorScheme[4]}; }
.day:hover::after {
    content: attr(data-count) " files (" attr(data-date) ")";
    position: absolute;
    bottom: 100%;
    left: 50%;
    transform: translateX(-50%);
    background: #333;
    color: white;
    padding: 4px 8px;
    border-radius: 4px;
    white-space: nowrap;
    z-index: 10;
    font-size: 11px;
    pointer-events: none;
}
.legend {
    display: flex;
    align-items: center;
    gap: 4px;
    margin-top: 8px;
    font-size: 10px;
    color: #4384A3;
}
.legend-box {
    width: 10px;
    height: 10px;
    border-radius: 2px;
}
</style>

<div class="heatmap-wrapper">
    <div class="heatmap-title">${year} Note Activity</div>
    <div class="heatmap-months">
        ${monthPositions.map(m => 
            `<span class="month-label" style="left: ${m.weekIndex * 13 + 5}px;">${monthNames[m.month]}</span>`
        ).join('')}
    </div>
    <div class="heatmap-body">
        <div class="weekday-labels">
            <span class="weekday-label">Mon</span>
            <span class="weekday-label">Tue</span>
            <span class="weekday-label">Wed</span>
            <span class="weekday-label">Thu</span>
            <span class="weekday-label">Fri</span>
            <span class="weekday-label">Sat</span>
            <span class="weekday-label">Sun</span>
        </div>
        <div class="heatmap">
            ${weeks.map(week => `
                <div class="week">
                    ${week.map(day => day ? `
                        <div class="day" 
                             data-level="${day.level}"
                             data-date="${day.date}"
                             data-count="${day.count}"
                             title="${day.date}: ${day.count} files">
                        </div>
                    ` : '<div class="day" style="background: transparent;"></div>').join('')}
                </div>
            `).join('')}
        </div>
    </div>
    <div class="legend">
        Less
        <div class="legend-box" style="background: ${colorScheme[0]};"></div>
        <div class="legend-box" style="background: ${colorScheme[1]};"></div>
        <div class="legend-box" style="background: ${colorScheme[2]};"></div>
        <div class="legend-box" style="background: ${colorScheme[3]};"></div>
        <div class="legend-box" style="background: ${colorScheme[4]};"></div>
        More
    </div>
</div>
`;
```
```dataviewjs
const allInsights = dv.pages("#insight-list")
  .flatMap(p => p.insights || [])
  .filter(i => i && i.text)
  .map(i => i.text);

if (allInsights.length === 0) {
  dv.paragraph("*No insights found*");
} else {
  const random = allInsights.sort(() => 0.5 - Math.random())[0];
  dv.paragraph(`<div style="text-align:center;color:#4384A3;font-size:0.95em;font-style:italic;padding:4px 0;margin:0;line-height:1.3">" ${random} "</div>`);
}
```
```dataviewjs
// 配置区
var DAILY_FOLDER = "Resources/DearDiary";
var CHART_DISPLAY_SIZE = 24;
var CHART_RESOLUTION = 1;
var CANVAS_SIZE = CHART_DISPLAY_SIZE * CHART_RESOLUTION;
var CUTOUT_PERCENT = "65%";

var HABIT_COLORS = {
    study: "#F0B87E",
    diet: "#7A9F8E",
    sleep: "#E8A0A0",
    default: "#607D8B"
};



// 日志预览提取函数
function extractOneMomentToday(text) {
    var lines = text.split("\n");
    var result = [];
    var inSection = false;

    for (var i = 0; i < lines.length; i++) {
        var line = lines[i];
        if (line.trim().indexOf("### ✨ One Moment Today") !== -1) {
            inSection = true;
            continue;
        }
        if (inSection) {
            if (line.trim().match(/^#{1,6}\s+/)) break;
            if (line.trim() === "") continue;
            if (line.trim() === "---" || line.trim() === "***") continue;
            result.push(line);
            if (result.length >= 5) break;
        }
    }
    return result.join("\n");
}

function cleanText(text) {
    return text.replace(/\[\[(.*?)\]\]/g, "$1")
               .replace(/\[(.*?)\]\((.*?)\)/g, "$1")
               .replace(/#[\w-]+/g, "")  // 移除标签，如 #habit/study
               .replace(/[\*~`]/g, "")
               .trim();
}


function getPreview(content) {
    var preview = extractOneMomentToday(content);
    if (preview) {
        var lines = preview.split("<br>").slice(0, 5);
        return cleanText(lines.join("\n"));
    }

    // 如果没有One Moment Today章节，取前几条非空非标题行
    var allLines = content.split("\n");
    var result = [];
    for (var i = 0; i < allLines.length; i++) {
        var line = allLines[i];
        var trimmed = line.trim();
        if (!trimmed) continue;
        if (trimmed.startsWith("#")) continue;
        if (trimmed === "---" || trimmed === "***") continue;
        // 保留原始换行，去掉标签后保留换行
        var cleaned = cleanText(line);
        if (cleaned) {
            result.push(cleaned);
            if (result.length >= 5) break;
        }
    }
    return result.join("\n");
}

// 今日与上月数据
var todayStr = window.moment().format("YYYY-MM-DD");
var todayPath = DAILY_FOLDER + "/" + todayStr + ".md";
var todayFile = app.vault.getAbstractFileByPath(todayPath);
var todayPreview = "";
var todayExists = false;

if (todayFile) {
    todayExists = true;
    var todayContent = await app.vault.cachedRead(todayFile);
    todayPreview = getPreview(todayContent);
    if (!todayPreview) todayPreview = "No journal entry yet.";
}

var pastDateStr = window.moment().subtract(1, "months").format("YYYY-MM-DD");
var pastPath = DAILY_FOLDER + "/" + pastDateStr + ".md";
var pastFile = app.vault.getAbstractFileByPath(pastPath);
var pastPreview = "";
var pastExists = false;

if (pastFile) {
    pastExists = true;
    var pastContent = await app.vault.cachedRead(pastFile);
    pastPreview = getPreview(pastContent);
    if (!pastPreview) pastPreview = "记忆空空...";
}

// 习惯打卡数据
var hToday = moment();
var hStartDate = hToday.clone().subtract(6, "days");

var hPages = dv.pages("#dearDiary").where(function(p) {
    var pageDate = moment(p.file.name, "YYYY-MM-DD", true);
    return pageDate.isValid() && pageDate.isSameOrAfter(hStartDate, "day") && pageDate.isSameOrBefore(hToday, "day");
});

var hPageMap = {};
for (var p of hPages) {
    var ds = moment(p.file.name, "YYYY-MM-DD").format("YYYY-MM-DD");
    hPageMap[ds] = p;
}

var hDateRange = [];
for (var i = 0; i < 7; i++) {
    hDateRange.push(hStartDate.clone().add(i, "days").format("YYYY-MM-DD"));
}

var habitRecords = {};
var allHabitKeys = new Set();

for (var d = 0; d < hDateRange.length; d++) {
    var ds = hDateRange[d];
    var page = hPageMap[ds];
    if (!page) continue;
    var content = await dv.io.load(page.file.path);
    var lines = content.split("\n");
    
    for (var li = 0; li < lines.length; li++) {
        var line = lines[li];
        // 匹配任务项
        var taskMatch = line.match(/^[\s]*[-*+][\s]+\[([ x])\]/);
        if (!taskMatch) continue;
        var rest = line.slice(taskMatch[0].length);
        
        // 匹配习惯标签 #habit/习惯名（后面可能有 -数值，也可能没有）
        var habitRegex = /#habit\/(\w+)(?:-\d+)?/g;
        var habitTags = rest.matchAll(habitRegex);
        
        for (var tagMatch of habitTags) {
            var key = tagMatch[1];
            allHabitKeys.add(key);
            if (!habitRecords[key]) habitRecords[key] = {};
            habitRecords[key][ds] = true;
        }
    }
}

// 计算习惯完成率
var chartData = [];
for (var key of allHabitKeys) {
    var completed = 0;
    var records = habitRecords[key] || {};
    for (var di = 0; di < hDateRange.length; di++) {
        if (records[hDateRange[di]]) completed++;
    }
    var rate = Math.round((completed / 7) * 100);
    chartData.push({ 
        rate: rate, 
        completed: completed, 
        habitKey: key,
        total: 7
    });
}

// 加载Chart.js依赖
if (typeof Chart === 'undefined') {
    await new Promise((resolve, reject) => {
        const s = document.createElement('script');
        s.src = 'https://cdn.jsdelivr.net/npm/chart.js';
        s.defer = true;
        s.async = true;
        s.onload = resolve;
        s.onerror = reject;
        document.head.appendChild(s);
    });
}

// 渲染三栏仪表板
var container = dv.el("div", "");
var h = "";

h += "<style>";
h += ".tcv{display:flex;gap:8px;margin:6px 10px;box-sizing:border-box;}";
h += ".tcv .tc{border-radius:8px;padding:8px 10px;box-sizing:border-box;}";
h += ".tcv .tc-today{flex:1.2;background:#EBE4E2;}";
h += ".tcv .tc-past{flex:1.2;background:#EBE4E2;}";
h += ".tcv .tc-habits{flex:0.5;background:transparent;padding:4px 8px;}";
h += ".tc-title{font-weight:bold;margin-bottom:4px;font-size:0.9em;color:#4384A3;}";
h += ".tc-date{font-size:0.7em;color:#999;margin-bottom:3px;}";
h += ".tc-preview{font-size:0.8em;line-height:1.4;color:var(--text-normal);max-height:80px;overflow:hidden;}";
h += ".tc-link{font-size:0.7em;margin-top:4px;}";
h += ".tc-link a{color:#4384A3;text-decoration:none;}";
h += ".tc-empty{font-size:0.8em;color:#999;font-style:italic;}";

// 习惯打卡样式
h += ".habits-container{margin-top:6px;}";
h += ".habit-item{margin:5px 0;display:flex;align-items:center;gap:4px;}";
h += ".habit-ring{width:24px;height:24px;}";
h += ".habit-label{font-size:0.75em;color:#4384A3;font-weight:500;text-transform:capitalize;}";
h += ".habit-stats{font-size:0.7em;color:#666;}";
h += ".habit-empty{font-size:0.8em;color:#999;font-style:italic;}";
h += "</style>";

h += "<div class='tcv'>";

// 左栏：今日日志
h += "<div class='tc tc-today'>";
h += "<div class='tc-title'>Today</div>";
h += "<div class='tc-date'>" + todayStr + "</div>";
if (todayExists) {
    h += "<div class='tc-preview'>" + todayPreview.replace(/\n/g, "<br>") + "</div>";
    h += "<div class='tc-link'><a data-href='" + todayPath + "' class='internal-link'>→ Full Note</a></div>";
} else {
    h += "<div class='tc-empty'>(How was today?)</div>";
}
h += "</div>";

// 中栏：上月今日
h += "<div class='tc tc-past'>";
h += "<div class='tc-title'>Last Month</div>";
h += "<div class='tc-date'>" + pastDateStr + "</div>";
if (pastExists) {
    h += "<div class='tc-preview'>" + pastPreview.replace(/\n/g, "<br>") + "</div>";
    h += "<div class='tc-link'><a data-href='" + pastPath + "' class='internal-link'>→ Full Note →</a></div>";
} else {
    h += "<div class='tc-empty'>(记忆空空...)</div>";
}
h += "</div>";

// 右栏：习惯打卡环图
h += "<div class='tc tc-habits'>";
h += "<div class='tc-title'>Habits</div>";

if (chartData.length == 0) {
    h += "<div class='habit-empty'>暂无习惯数据</div>";
} else {
    h += "<div class='habits-container'>";
    for (var ci = 0; ci < chartData.length; ci++) {
        var item = chartData[ci];
        var chartId = "habit-ring-" + ci;
        var color = HABIT_COLORS[item.habitKey] || HABIT_COLORS["default"];

        h += "<div class='habit-item'>";
        h += "<canvas id='" + chartId + "' class='habit-ring' width='" + CANVAS_SIZE + "' height='" + CANVAS_SIZE + "'></canvas>";
        h += "<div class='habit-label'>" + item.habitKey + " ✓" + item.completed + "</div>";
        h += "</div>";
    }
    h += "</div>";
}

h += "</div>"; // 结束tc-habits
h += "</div>"; // 结束tcv

container.innerHTML = h;

// 渲染习惯打卡环图
setTimeout(function() {
    for (var ci = 0; ci < chartData.length; ci++) {
        var item = chartData[ci];
        var chartId = "habit-ring-" + ci;
        var canvas = container.querySelector("#" + chartId);
        if (!canvas) continue;
        
        var ctx = canvas.getContext("2d");
        var color = HABIT_COLORS[item.habitKey] || HABIT_COLORS["default"];
        
        new Chart(ctx, {
            type: "doughnut",
            data: {
                datasets: [{
                    data: [item.rate, 100 - item.rate],
                    backgroundColor: [color, "#E0E0E0"],
                    borderWidth: 0
                }]
            },
            options: {
                responsive: false,
                maintainAspectRatio: false,
                cutout: CUTOUT_PERCENT,
                plugins: {
                    legend: { display: false },
                    tooltip: { enabled: false }
                }
            }
        });
    }
}, 300);
```
```dataviewjs 
// ┌──────────────────────────────────────┐
// │         📦 加载依赖                    │
// └──────────────────────────────────────┘

if (typeof Chart === 'undefined') {
    const s = document.createElement('script');
    s.src = 'https://cdn.jsdelivr.net/npm/chart.js@4.4.0/dist/chart.umd.min.js';
    document.head.appendChild(s);
    await new Promise(resolve => {
        s.onload = resolve;
        s.onerror = () => resolve();
    });
}


// ┌──────────────────────────────────────┐
// │            🎨 配置区                  │
// └──────────────────────────────────────┘

var CHART_DISPLAY_SIZE = 24;
var CHART_RESOLUTION = 2;
var CANVAS_SIZE = CHART_DISPLAY_SIZE * CHART_RESOLUTION;
var CUTOUT_PERCENT = "65%";

// ┌──────────────────────────────────────┐
// │         📊 项目进度数据                │
// └──────────────────────────────────────┘

var allProjects = dv.pages("#project-list")
    .flatMap(function(p) {
        return (p.projects || []).map(function(proj) {
            return { name: proj.name, status: proj.status, priority: proj.priority, progress: proj.progress, source: p.file.name };
        });
    })
    .filter(function(p) { return p && p.name; });

var statusIcon = { active: "▶", paused: "⏸", done: "✓", backlog: "○" };
var prioOrder = { high: 3, medium: 2, low: 1 };

var activeProjects = allProjects.filter(function(p) { return p.status !== "done"; });
activeProjects.sort(function(a, b) {
    return (prioOrder[b.priority || "low"] || 0) - (prioOrder[a.priority || "low"] || 0);
});

function makeProgressBar(pct) {
    var val = pct || 0;
    var filled = Math.round(val / 10);
    var empty = 10 - filled;
    var result = "";
    for (var i = 0; i < filled; i++) {
        result = result + "<span style='color:#6B9E8A;font-size:0.85em'>█</span>";
    }
    for (var i = 0; i < empty; i++) {
        result = result + "<span style='color:#E6E9DC;font-size:0.85em'>█</span>";
    }
    result = result + " <span style='color:#4384A3;font-size:0.8em'>" + val + "%</span>";
    return result;
}

// ┌──────────────────────────────────────┐
// │         🌱 习惯打卡数据                │
// └──────────────────────────────────────┘

var hToday = moment();
var hStartDate = hToday.clone().subtract(6, "days");

var hPages = dv.pages("#dearDiary").where(function(p) {
    var pageDate = moment(p.file.name, "YYYY-MM-DD", true);
    return pageDate.isValid() && pageDate.isSameOrAfter(hStartDate, "day") && pageDate.isSameOrBefore(hToday, "day");
});

var hPageMap = {};
for (var p of hPages.values) {
    var ds = moment(p.file.name, "YYYY-MM-DD").format("YYYY-MM-DD");
    hPageMap[ds] = p;
}

var hDateRange = [];
for (var i = 0; i < 7; i++) {
    hDateRange.push(hStartDate.clone().add(i, "days").format("YYYY-MM-DD"));
}

var habitRecords = {};
var allHabitKeys = new Set();

for (var d = 0; d < hDateRange.length; d++) {
    var ds = hDateRange[d];
    var page = hPageMap[ds];
    if (!page) continue;
    var content = await dv.io.load(page.file.path);
    var lines = content.split("\n");
    for (var li = 0; li < lines.length; li++) {
        var line = lines[li];
        var taskMatch = line.match(/^- \[\s*([xX])\s*\]\s*(.+)$/);
        if (!taskMatch) continue;
        var rest = taskMatch[2];
        var habitTags = rest.matchAll(/#habit\/([a-zA-Z0-9_-]+)/g);
        for (var tagMatch of habitTags) {
            var key = tagMatch[1];
            allHabitKeys.add(key);
            if (!habitRecords[key]) habitRecords[key] = {};
            habitRecords[key][ds] = true;
        }
    }
}

var chartData = [];
for (var key of allHabitKeys) {
    var completed = 0;
    var records = habitRecords[key] || {};
    for (var di = 0; di < hDateRange.length; di++) {
        if (records[hDateRange[di]]) completed = completed + 1;
    }
    var rate = Math.round((completed / 7) * 100);
    chartData.push({ rate: rate, completed: completed, habitKey: key });
}

// ┌──────────────────────────────────────┐
// │          🖼️ 渲染双栏：项目 + 任务      │
// └──────────────────────────────────────┘

var container = dv.el("div", "");
var h = "";

h = h + "<style>";
h = h + ".ph-row{display:flex;gap:8px;margin:6px 0;align-items:flex-start;}";
h = h + ".ph-row .ph-col{border-radius:8px;padding:4px 8px;box-sizing:border-box;}";
h = h + ".ph-row .ph-proj{flex:4;background:transparent;}";
h = h + ".ph-row .ph-tasks{flex:0.8;background:transparent;}";
h = h + ".ph-row .ph-title{font-weight:bold;margin:0;padding:0;font-size:0.9em;color:#4384A3;line-height:1.3;}";

// 项目表格样式
h = h + ".ph-table{width:100%;border-collapse:collapse;font-size:0.9em;margin:0;padding:0;border-spacing:0;margin-top:-10px;}";
h = h + ".ph-table td{padding:2px 6px;color:#4384A3;font-size:0.9em;line-height:1.3;}";
h = h + ".ph-table tr:hover{background:rgba(67,132,163,0.05);}";

// 习惯样式 - 优化为更紧凑和均匀
h = h + ".ph-row .hi{margin:4px 0;display:flex;align-items:center;gap:4px;}";
h = h + ".ph-row .hi:first-child{margin-top:2px;}";
h = h + ".ph-row .hi-label{font-size:0.75em;margin-top:0;color:#4384A3;font-weight:500;}";

// Tasks样式 - 增加与标题的间距
h = h + ".tasks-container{margin-top:6px;}";  // 增加顶部间距到16px，整体下移
h = h + ".tasks-item{margin:5px 0;display:flex;align-items:center;gap:4px;}";
h = h + ".tasks-item:first-child{margin-top:0;}";
h = h + ".tasks-label{font-size:0.75em;margin-top:0;color:#4384A3;font-weight:500;}";
h = h + ".ph-empty{font-size:0.82em;color:#999;font-style:italic;}";
h = h + "</style>";


h = h + "<div class='ph-row'>";

// 左栏：项目进度
h = h + "<div class='ph-col ph-proj'>";
h = h + "<div class='ph-title'>Projects</div>";

if (activeProjects.length === 0) {
    h = h + "<div class='ph-empty'>No ongoing projects</div>";
} else {
    h = h + "<table class='ph-table'>";
    for (var pi = 0; pi < activeProjects.length; pi++) {
        var proj = activeProjects[pi];
        var icon = statusIcon[proj.status] || "⚪";
        h = h + "<tr>";
        h = h + "<td>" + (proj.name || "") + "</td>";
        h = h + "<td>" + icon + " " + (proj.status || "unknown") + "</td>";
        h = h + "<td>" + (proj.priority || "-") + "</td>";
        h = h + "<td>" + makeProgressBar(proj.progress) + "</td>";
        h = h + "</tr>";
    }
    h = h + "</table>";
}
h = h + "</div>";

// 右栏：Tasks统计
h = h + "<div class='ph-col ph-tasks'>";
h = h + "<div class='ph-title'>Tasks</div>";

var todayStr = moment().format("YYYY-MM-DD");
var weekStart = moment().startOf('isoWeek').format("YYYY-MM-DD");
var monthStart = moment().startOf('month').format("YYYY-MM-DD");

var todayTasks = 0, weekTasks = 0, monthTasks = 0;

var workPages = dv.pages('"Projects"');

for (var page of workPages.values) {
    var fileContent = await dv.io.load(page.file.path);
    if (!fileContent) continue;
    var lines = fileContent.split("\n");
    for (var li = 0; li < lines.length; li++) {
        var line = lines[li];
        var m = line.match(/^-\s*\[x\]\s*.+✅\s*(\d{4}-\d{2}-\d{2})/);
        if (!m) continue;
        var d = m[1];
        if (d === todayStr) todayTasks++;
        if (d >= weekStart) weekTasks++;
        if (d >= monthStart) monthTasks++;
    }
}

var todayTarget = 5, weekTarget = 25, monthTarget = 100;

window._taskData = [
    { completed: todayTasks, target: todayTarget, label: "today", color: "#E7BA86" },
    { completed: weekTasks, target: weekTarget, label: "week", color: "#829E8F" },
    { completed: monthTasks, target: monthTarget, label: "month", color: "#DDA3A2" }
];

h = h + "<div class='tasks-container'>";
for (var ti = 0; ti < window._taskData.length; ti++) {
    var item = window._taskData[ti];
    var chartId = "ph-task-ring-" + ti;
    h = h + "<div class='tasks-item'>";
    h = h + "<canvas id='" + chartId + "' width='" + CANVAS_SIZE + "' height='" + CANVAS_SIZE + "' style='width:" + CHART_DISPLAY_SIZE + "px;height:" + CHART_DISPLAY_SIZE + "px;'></canvas>";
    h = h + "<div class='tasks-label'>" + item.label + " ✓" + item.completed + "</div>";
    h = h + "</div>";
}
h = h + "</div>";

h = h + "</div>";


container.innerHTML = h;

// 渲染Tasks环形图
setTimeout(function() {
    var taskData = window._taskData || [];
    for (var ti = 0; ti < taskData.length; ti++) {
        var item = taskData[ti];
        var chartId = "ph-task-ring-" + ti;
        var canvas = container.querySelector("#" + chartId);
        if (!canvas) continue;
        var ctx = canvas.getContext("2d");
        var rate = Math.min(Math.round((item.completed / item.target) * 100), 100);
        var bgColor = item.color;
        new Chart(ctx, {
            type: "doughnut",
            data: { datasets: [{ data: [rate, 100 - rate], backgroundColor: [bgColor, "#E0E0E0"], borderWidth: 0 }] },
            options: { responsive: false, maintainAspectRatio: false, cutout: CUTOUT_PERCENT, plugins: { legend: { display: false }, tooltip: { enabled: false } } }
        });
    }
}, 300);

```

> [!multi-column]
>> [!info] Today
>> ```tasks
>> not done
>> group by tags
>> hide backlink
>> hide tags
>> priority is high
>>tags do not include habit
>> limit 15
>> ```
>
>> [!note] Todo
>> ```tasks
>> not done
>> group by tags
>> hide backlink
>> hide tags
>> tags do not include habit
>> limit 15
>> ```
>
>> [!Check] Done
>> ```tasks
>> done
>> group by tags
>> hide backlink
>> hide tags
>> tags do not include habit
>> limit 15
>> ```
>

