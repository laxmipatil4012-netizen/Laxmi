<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>AITM Official Master Scheduler - Pro</title>
    <style>
        :root { --aitm-blue: #1e3a8a; --border-bold: #000000; }
        body { font-family: 'Times New Roman', serif; background: #e5e7eb; padding: 20px; color: #000; }
        .container { max-width: 1300px; margin: auto; background: white; padding: 30px; border: 1px solid #ccc; box-shadow: 0 0 15px rgba(0,0,0,0.1); }
        .official-header { text-align: center; border-bottom: 2px solid var(--border-bold); padding-bottom: 10px; margin-bottom: 20px; }
        
        /* Configuration Styles */
        .config-panel { background: #f3f4f6; padding: 20px; border: 1px solid #999; margin-bottom: 20px; }
        .mapping-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(300px, 1fr)); gap: 20px; margin-top: 15px; }
        .div-config-card { background: #fff; padding: 15px; border: 1px solid var(--aitm-blue); border-top: 5px solid var(--aitm-blue); }
        .div-config-card h3 { margin-top: 0; color: var(--aitm-blue); border-bottom: 1px solid #ddd; }
        
        .input-row { display: flex; justify-content: space-between; margin-bottom: 5px; align-items: center; font-size: 12px; }
        .input-row input { width: 60%; padding: 4px; border: 1px solid #ccc; }

        /* Table Styles */
        table { width: 100%; border-collapse: collapse; table-layout: fixed; margin-top: 10px; background: #fff; }
        th, td { border: 1px solid var(--border-bold); padding: 5px 2px; text-align: center; font-size: 11px; }
        th { background: #eeeeee; font-weight: bold; }
        
        .break-col { width: 25px; background: #f3f4f6; font-weight: bold; font-size: 10px; writing-mode: vertical-rl; text-orientation: upright; }
        .lab-session { background: #fffde7; font-weight: bold; color: #b45309; border: 2px solid #b45309 !important; }
        .teacher-tag { display: block; font-size: 9px; color: var(--aitm-blue); font-weight: bold; border-top: 1px dotted #ccc; margin-top: 2px; }
        
        .section-header { background: #333; color: #fff; padding: 10px; margin-top: 40px; text-align: center; text-transform: uppercase; }
        .division-title { background: var(--aitm-blue); color: white; padding: 5px 15px; margin-top: 25px; display: inline-block; font-weight: bold; }
        
        button { background: var(--aitm-blue); color: white; border: none; padding: 10px 20px; cursor: pointer; font-weight: bold; width: 100%; margin-top: 10px; }
        button:hover { background: #152a61; }

        .faculty-grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(300px, 1fr)); gap: 15px; margin-top: 20px; }
        .faculty-card { border: 1px solid #aaa; background: #fff; padding: 10px; }
    </style>
</head>
<body>

<div class="container">
    <div class="official-header">
        <h1>ANGADI INSTITUTE OF TECHNOLOGY AND MANAGEMENT</h1>
        <h2>Department of Computer Science & Engineering</h2>
    </div>

    <div class="config-panel">
        <div style="display:flex; gap: 20px; align-items: center; margin-bottom: 15px;">
            <label><b>1. Set Numbers:</b></label>
            Divisions: <input type="number" id="numDivs" value="2" style="width:50px" onchange="setupInputs()">
            Theory Subs: <input type="number" id="numSubs" value="5" style="width:50px" onchange="setupInputs()">
            Labs: <input type="number" id="numLabs" value="2" style="width:50px" onchange="setupInputs()">
        </div>
        
        <div id="mappingArea" class="mapping-grid">
            </div>
        
        <button onclick="generateSchedule()">GENERATE ALL TIME TABLES</button>
    </div>

    <div id="displayArea"></div>
    
    <div id="facultySection" style="display:none;">
        <div class="section-header">Faculty Wise Consolidated Load</div>
        <div id="facultyArea" class="faculty-grid"></div>
    </div>
</div>

<script>
// Initial setup on load
window.onload = setupInputs;

function setupInputs() {
    const nDivs = parseInt(document.getElementById("numDivs").value);
    const nSubs = parseInt(document.getElementById("numSubs").value);
    const nLabs = parseInt(document.getElementById("numLabs").value);
    const area = document.getElementById("mappingArea");
    area.innerHTML = "";

    for (let i = 0; i < nDivs; i++) {
        let divChar = String.fromCharCode(65 + i);
        let card = document.createElement("div");
        card.className = "div-config-card";
        let html = `<h3>Division ${divChar}</h3>`;
        
        html += `<b>Theory (Subject : Teacher)</b><br>`;
        for(let j=1; j<=nSubs; j++) {
            html += `<div class="input-row">
                <input type="text" id="sub_${divChar}_${j}" value="Sub${j}" placeholder="Sub Name">
                <input type="text" id="fac_${divChar}_${j}" value="Prof. Name" placeholder="Teacher">
            </div>`;
        }
        
        html += `<br><b>Labs (Lab Name : Teacher)</b><br>`;
        for(let k=1; k<=nLabs; k++) {
            html += `<div class="input-row">
                <input type="text" id="lab_${divChar}_${k}" value="Lab${k}" placeholder="Lab Name">
                <input type="text" id="labfac_${divChar}_${k}" value="Lab Prof" placeholder="Lab Teacher">
            </div>`;
        }
        
        card.innerHTML = html;
        area.appendChild(card);
    }
}

function generateSchedule() {
    const nDivs = parseInt(document.getElementById("numDivs").value);
    const nSubs = parseInt(document.getElementById("numSubs").value);
    const nLabs = parseInt(document.getElementById("numLabs").value);
    const days = ["Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday"];
    const timeSlots = ["9:00-10:00", "10:00-11:00", "11:15-12:15", "12:15-1:15", "2:00-3:00", "3:00-4:00", "4:00-5:00"];
    
    let html = "";
    let teacherDutyMap = {};

    for (let d = 0; d < nDivs; d++) {
        let divChar = String.fromCharCode(65 + d);
        let divName = "Division " + divChar;
        
        // Collect data for this division
        let myTheory = [];
        for(let j=1; j<=nSubs; j++) {
            myTheory.push({ 
                sub: document.getElementById(`sub_${divChar}_${j}`).value, 
                fac: document.getElementById(`fac_${divChar}_${j}`).value 
            });
        }
        let myLabs = [];
        for(let k=1; k<=nLabs; k++) {
            myLabs.push({ 
                sub: document.getElementById(`lab_${divChar}_${k}`).value, 
                fac: document.getElementById(`labfac_${divChar}_${k}`).value 
            });
        }

        html += `<div class="division-title">DEPARTMENT OF CSE - ${divName}</div>
        <table>
            <tr>
                <th style="width:70px;">DAY</th>
                <th>1<br>${timeSlots[0]}</th><th>2<br>${timeSlots[1]}</th>
                <th class="break-col">TEA</th>
                <th>3<br>${timeSlots[2]}</th><th>4<br>${timeSlots[3]}</th>
                <th class="break-col">LUNCH</th>
                <th>5<br>${timeSlots[4]}</th><th>6<br>${timeSlots[5]}</th><th>7<br>${timeSlots[6]}</th>
            </tr>`;

        days.forEach((day, dayIdx) => {
            html += `<tr><td><b>${day.toUpperCase()}</b></td>`;
            
            if (day === "Saturday") {
                // Saturday: 2 slots
                for(let s=0; s<2; s++) {
                    let item = myTheory[s % myTheory.length];
                    html += `<td>${item.sub}<span class="teacher-tag">${item.fac}</span></td>`;
                    addDuty(teacherDutyMap, item.fac, day, divName, timeSlots[s], item.sub);
                }
                html += `<td class="break-col">-</td><td colspan="6" style="color:#888; font-style:italic">Internal Assessment / Library</td>`;
            } else {
                // Mon-Fri
                let labToday = (dayIdx < myLabs.length) ? myLabs[dayIdx] : null;
                
                for(let s=0; s<7; s++) {
                    // Place Labs in slots 3-4 (s=2,3)
                    if(labToday && s === 2) {
                        html += `<td colspan="2" class="lab-session">LAB: ${labToday.sub}<br><span class="teacher-tag">${labToday.fac}</span></td>`;
                        addDuty(teacherDutyMap, labToday.fac, day, divName, `${timeSlots[2]}-${timeSlots[3]}`, "LAB: "+labToday.sub);
                        s++; // skip slot 4
                    } else {
                        let tIdx = (dayIdx + s) % myTheory.length;
                        let item = myTheory[tIdx];
                        html += `<td>${item.sub}<span class="teacher-tag">${item.fac}</span></td>`;
                        addDuty(teacherDutyMap, item.fac, day, divName, timeSlots[s], item.sub);
                    }
                    
                    if(s === 1) html += `<td class="break-col">TEA</td>`;
                    if(s === 3) html += `<td class="break-col">LUNCH</td>`;
                }
            }
            html += `</tr>`;
        });
        html += `</table>`;
    }

    document.getElementById("displayArea").innerHTML = html;
    renderFacultyChart(teacherDutyMap, days);
}

function addDuty(map, teacher, day, div, slot, sub) {
    if(!teacher) return;
    if(!map[teacher]) map[teacher] = [];
    map[teacher].push({ day, div, slot, sub });
}

function renderFacultyChart(map, days) {
    document.getElementById("facultySection").style.display = "block";
    let fHtml = "";
    Object.keys(map).sort().forEach(t => {
        fHtml += `<div class="faculty-card"><div style="background:#444; color:white; padding:5px; font-weight:bold; margin-bottom:5px;">${t}</div><table style="font-size:9px;">`;
        days.forEach(day => {
            let tasks = map[t].filter(task => task.day === day);
            let desc = tasks.map(task => `${task.slot}: ${task.sub} (${task.div})`).join("<br>") || "-";
            if(desc !== "-") fHtml += `<tr><td style="width:40px;">${day.substr(0,3)}</td><td>${desc}</td></tr>`;
        });
        fHtml += `</table></div>`;
    });
    document.getElementById("facultyArea").innerHTML = fHtml;
}
</script>

</body>
</html>
