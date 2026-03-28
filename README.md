# techspark

from flask import Flask, render_template_string

app = Flask(__name__)

EMISSION_FACTOR = 0.82

HTML = """
<!DOCTYPE html>
<html>
<head>
<title>EnergyLens</title>
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>

<style>
body {
    font-family: Arial;
    background: #f4f6f9;
    padding: 20px;
}

h1 {
    color: #2c3e50;
}

.card {
    background: white;
    padding: 20px;
    margin: 10px 0;
    border-radius: 10px;
    box-shadow: 0 4px 8px rgba(0,0,0,0.1);
}

input, select, button {
    padding: 10px;
    margin: 5px;
}

button {
    background: #3498db;
    color: white;
    border: none;
    cursor: pointer;
    border-radius: 5px;
}

.danger {
    color: red;
    font-weight: bold;
}
</style>
</head>

<body>

<h1>EnergyLens</h1>

<div class="card">
<h2>Add Appliance</h2>

<select id="room">
    <option>Living Room</option>
    <option>Bedroom</option>
    <option>Kitchen</option>
</select>

<input id="name" placeholder="Appliance Name">
<input id="watt" placeholder="Watt">
<input id="hours" placeholder="Hours per day">

<button onclick="addAppliance()">Add</button>

<br>

<button onclick="scanRoom()">Scan Room (Demo AI)</button>
<button onclick="scanWatt()">Scan Watt</button>
</div>

<div class="card">
<h2>Electricity Rate</h2>
<input id="rate" value="8">
</div>

<div class="card">
<h2>Appliance List</h2>
<ul id="list"></ul>
</div>

<div class="card">
<h2>Dashboard</h2>
<p id="daily"></p>
<p id="monthly"></p>
<p id="yearly"></p>
<p id="co2"></p>
<p id="top"></p>
<canvas id="chart"></canvas>
</div>

<div class="card">
<h2>What-if Simulator</h2>
<input id="newHours" placeholder="New Hours">
<button onclick="simulate()">Simulate</button>
<p id="saving"></p>
</div>

<div class="card">
<h2>Smart Recommendation</h2>
<p id="suggestion"></p>
</div>

<script>

let appliances = [];

function addAppliance() {
    let room = document.getElementById("room").value;
    let name = document.getElementById("name").value;
    let watt = parseFloat(document.getElementById("watt").value);
    let hours = parseFloat(document.getElementById("hours").value);

    if (!name || isNaN(watt) || isNaN(hours)) return;

    appliances.push({room, name, watt, hours});
    render();
}

function scanRoom() {
    appliances.push(
        {room:"Living Room", name:"TV", watt:150, hours:5},
        {room:"Bedroom", name:"Fan", watt:75, hours:8},
        {room:"Bedroom", name:"Laptop", watt:60, hours:6}
    );
    render();
}

function scanWatt() {
    document.getElementById("watt").value = 120;
}

function render() {

    let rate = parseFloat(document.getElementById("rate").value);
    let list = document.getElementById("list");
    list.innerHTML = "";

    let totalDaily = 0;
    let totalMonthly = 0;
    let totalYearly = 0;

    let data = [];
    let labels = [];

    appliances.forEach(a => {

        let daily = (a.watt * a.hours)/1000;
        let monthly = daily * 30;
        let yearly = daily * 365;

        let cost = yearly * rate;

        totalDaily += daily;
        totalMonthly += monthly;
        totalYearly += yearly;

        let li = document.createElement("li");
        li.innerHTML = a.room + " - " + a.name + " : ₹" + cost.toFixed(0);
        list.appendChild(li);

        data.push(cost);
        labels.push(a.name);
    });

    document.getElementById("daily").innerHTML =
        "Daily Units: " + totalDaily.toFixed(2) + " kWh";

    document.getElementById("monthly").innerHTML =
        "Monthly Units: " + totalMonthly.toFixed(2) + " kWh";

    document.getElementById("yearly").innerHTML =
        "Yearly Cost: ₹" + (totalYearly * rate).toFixed(0);

    document.getElementById("co2").innerHTML =
        "CO2 Emission: " + (totalYearly * 0.82).toFixed(2) + " kg";

    if (data.length > 0) {
        let max = Math.max(...data);
        let idx = data.indexOf(max);

        document.getElementById("top").innerHTML =
            "Highest Consumption: <span class='danger'>" +
            appliances[idx].name + "</span>";

        document.getElementById("suggestion").innerHTML =
            "Consider reducing usage of " + appliances[idx].name +
            " to save electricity.";
    }

    if (window.chart) {
        window.chart.destroy();
    }

    window.chart = new Chart(document.getElementById("chart"), {
        type: 'bar',
        data: {
            labels: labels,
            datasets: [{ data: data }]
        }
    });
}

function simulate() {

    let newHours = parseFloat(document.getElementById("newHours").value);
    let rate = parseFloat(document.getElementById("rate").value);

    let oldCost = 0;
    let newCost = 0;

    appliances.forEach(a => {
        let oldYear = (a.watt * a.hours * 365)/1000;
        let newYear = (a.watt * newHours * 365)/1000;

        oldCost += oldYear * rate;
        newCost += newYear * rate;
    });

    let saving = oldCost - newCost;

    document.getElementById("saving").innerHTML =
        "Estimated Yearly Saving: ₹" + saving.toFixed(0);
}

</script>

</body>
</html>
"""

@app.route("/")
def home():
    return render_template_string(HTML)

if __name__ == "__main__":
    app.run(debug=True)
