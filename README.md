<!DOCTYPE html>
<html lang="bn">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>BD BET - Ghost Admin Pro</title>
    <style>
        :root { --bg: #0b0e11; --card: #1e2329; --accent: #fcd535; --green: #0ecb81; --red: #ff3e3e; }
        body { background: var(--bg); color: white; font-family: 'Segoe UI', sans-serif; margin: 0; user-select: none; }
        
        header { background: #181a20; padding: 15px; display: flex; justify-content: space-between; align-items: center; border-bottom: 1px solid #333; }
        .logo { color: var(--accent); font-size: 20px; font-weight: bold; cursor: crosshair; }
        .balance-box { background: #2b3139; padding: 5px 12px; border-radius: 5px; color: var(--green); }

        /* Hidden Admin Overlay */
        #admin-overlay { display: none; position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: #000; z-index: 9999; padding: 20px; box-sizing: border-box; overflow-y: auto; }
        .admin-card { background: var(--card); padding: 15px; border-radius: 10px; margin-bottom: 15px; border: 1px solid var(--accent); }
        
        table { width: 100%; border-collapse: collapse; margin-top: 10px; font-size: 12px; }
        th, td { border: 1px solid #444; padding: 8px; text-align: left; }
        th { background: #333; color: var(--accent); }

        .btn { border: none; padding: 10px; border-radius: 5px; font-weight: bold; cursor: pointer; width: 100%; margin-top: 5px; }
        .btn-green { background: var(--green); color: white; }
        .btn-close { background: var(--red); color: white; width: auto; padding: 5px 15px; }

        .page { display: none; padding: 15px; }
        .active { display: block; }
        .grid { display: grid; grid-template-columns: 1fr 1fr; gap: 10px; }
        .game-card { background: var(--card); padding: 20px; border-radius: 10px; text-align: center; border: 1px solid #333; }
    </style>
</head>
<body>

<header>
    <div class="logo" id="secret-trigger">BD BET</div>
    <div class="balance-box">৳ <span id="main-bal">500</span></div>
</header>

<div id="lobby" class="page active">
    <h3>পপুলার গেম</h3>
    <div class="grid">
        <div class="game-card" onclick="alert('গেম লোড হচ্ছে...')">🚀 Aviator</div>
        <div class="game-card" onclick="alert('গেম লোড হচ্ছে...')">🃏 Super Ace</div>
    </div>
</div>

<div id="admin-overlay">
    <button class="btn-close" onclick="closeAdmin()">বন্ধ করুন (X)</button>
    <h2 style="color:var(--accent)">অ্যাডমিন ড্যাশবোর্ড</h2>
    
    <div class="admin-card">
        <h3>সব রেজিস্টার্ড ইউজার লিস্ট</h3>
        <table>
            <thead>
                <tr>
                    <th>নাম/ID</th>
                    <th>ফোন নম্বর</th>
                    <th>ব্যালেন্স</th>
                    <th>অ্যাকশন</th>
                </tr>
            </thead>
            <tbody id="user-list">
                </tbody>
        </table>
    </div>

    <div class="admin-card">
        <h3>ব্যালেন্স আপডেট (ম্যানুয়াল)</h3>
        <input type="text" id="target-user" placeholder="ইউজার আইডি দিন" style="width:90%; padding:10px; margin:5px 0;">
        <input type="number" id="mod-amt" placeholder="টাকার পরিমাণ (+ বা -)" style="width:90%; padding:10px; margin:5px 0;">
        <button class="btn btn-green" onclick="manualUpdate()">আপডেট করুন</button>
    </div>
</div>

<script>
    // ডামি ইউজার ডাটাবেস (বাস্তবে এটি সার্ভার থেকে আসে)
    let users = [
        {id: "user01", phone: "01700000000", bal: 500},
        {id: "user02", phone: "01800000000", bal: 1200},
        {id: "emran_boss", phone: "01900000000", bal: 5000}
    ];

    let myBalance = 500;
    const adminPass = "emran200";

    function updateUI() {
        document.getElementById('main-bal').innerText = myBalance;
        renderUsers();
    }

    // --- অ্যাডমিন ঢোকার গোপন পদ্ধতি ---
    let trigger = document.getElementById('secret-trigger');
    let pressTimer;
    let clickCount = 0;

    trigger.addEventListener('mousedown', () => { 
        pressTimer = setTimeout(() => {
            alert("মোড সক্রিয়! এবার ৩ বার লোগোতে ক্লিক করুন।");
            enableClickTrigger();
        }, 5000); // ৫ সেকেন্ড চেপে ধরে রাখতে হবে
    });

    trigger.addEventListener('mouseup', () => clearTimeout(pressTimer));

    function enableClickTrigger() {
        trigger.onclick = () => {
            clickCount++;
            if(clickCount >= 3) {
                let p = prompt("সিক্রেট পাসওয়ার্ড দিন:");
                if(p === adminPass) {
                    document.getElementById('admin-overlay').style.display = 'block';
                }
                clickCount = 0;
            }
        };
    }

    function closeAdmin() {
        document.getElementById('admin-overlay').style.display = 'none';
    }

    // ইউজার লিস্ট রেন্ডার করা
    function renderUsers() {
        const list = document.getElementById('user-list');
        list.innerHTML = "";
        users.forEach(u => {
            list.innerHTML += `
                <tr>
                    <td>${u.id}</td>
                    <td>${u.phone}</td>
                    <td>৳${u.bal}</td>
                    <td><button onclick="editUser('${u.id}')" style="background:orange; border:none; border-radius:3px;">Edit</button></td>
                </tr>
            `;
        });
    }

    function editUser(id) {
        let user = users.find(u => u.id === id);
        let newBal = prompt(`${id} এর নতুন ব্যালেন্স কত হবে?`, user.bal);
        if(newBal !== null) {
            user.bal = parseFloat(newBal);
            renderUsers();
            alert("আপডেট হয়েছে!");
        }
    }

    function manualUpdate() {
        let id = document.getElementById('target-user').value;
        let amt = parseFloat(document.getElementById('mod-amt').value);
        let user = users.find(u => u.id === id);
        if(user) {
            user.bal += amt;
            renderUsers();
            alert("সফল হয়েছে!");
        } else {
            alert("ইউজার পাওয়া যায়নি!");
        }
    }

    updateUI();
</script>
</body>
</html>
