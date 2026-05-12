# Digitark OÜ — Siseveebi leht

Kopeeri allolev kood faili `C:\inetpub\wwwroot\index.html` serveris.

> **Eeltingimus:** IIS roll peab olema paigaldatud ja teenus töötamas enne faili kopeerimist.

```html
<!DOCTYPE html>
<html lang="et">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Digitark OÜ — Siseveeb</title>
    <style>
        * { box-sizing: border-box; margin: 0; padding: 0; }

        body {
            font-family: Arial, sans-serif;
            background-color: #eef1f5;
            color: #333;
        }

        .header {
            background: linear-gradient(135deg, #002855, #004b99);
            color: white;
            padding: 18px 40px;
            display: flex;
            align-items: center;
            justify-content: space-between;
            box-shadow: 0 2px 8px rgba(0,0,0,0.25);
        }
        .header-left h1 { font-size: 22px; letter-spacing: 1px; }
        .header-left p { font-size: 12px; color: #a0c0e8; margin-top: 2px; }
        .header-right { text-align: right; }
        #clock { font-size: 26px; font-weight: bold; letter-spacing: 2px; }
        #date { font-size: 12px; color: #a0c0e8; margin-top: 2px; }

        .nav {
            background: #003580;
            display: flex;
            gap: 2px;
            padding: 0 40px;
        }
        .nav a {
            color: #cde;
            text-decoration: none;
            padding: 10px 18px;
            font-size: 13px;
            transition: background 0.2s;
            cursor: pointer;
        }
        .nav a:hover, .nav a.active { background: #0050b3; color: white; }

        .container {
            max-width: 1000px;
            margin: 30px auto;
            padding: 0 20px;
            display: grid;
            grid-template-columns: 2fr 1fr;
            gap: 20px;
        }

        .card {
            background: white;
            border-radius: 8px;
            padding: 24px;
            box-shadow: 0 1px 6px rgba(0,0,0,0.08);
            animation: fadeIn 0.4s ease;
        }

        @keyframes fadeIn {
            from { opacity: 0; transform: translateY(10px); }
            to   { opacity: 1; transform: translateY(0); }
        }

        .card h2 {
            font-size: 15px;
            color: #002855;
            border-bottom: 2px solid #e0e8f0;
            padding-bottom: 10px;
            margin-bottom: 16px;
            text-transform: uppercase;
            letter-spacing: 0.5px;
        }

        .notice {
            border-left: 4px solid #004b99;
            padding: 10px 14px;
            margin-bottom: 12px;
            background: #f0f5ff;
            border-radius: 0 6px 6px 0;
            cursor: pointer;
            transition: background 0.2s;
        }
        .notice:hover { background: #e0ecff; }
        .notice .notice-title {
            font-weight: bold;
            font-size: 13px;
            color: #002855;
            display: flex;
            justify-content: space-between;
        }
        .notice .notice-date { font-size: 11px; color: #888; margin-top: 2px; }
        .notice .notice-body {
            font-size: 13px;
            color: #555;
            margin-top: 8px;
            display: none;
            line-height: 1.5;
        }
        .notice.open .notice-body { display: block; }
        .notice .arrow { transition: transform 0.2s; }
        .notice.open .arrow { transform: rotate(90deg); }

        .quicklinks { display: flex; flex-direction: column; gap: 10px; }
        .ql-btn {
            background: #f0f5ff;
            border: 1px solid #cde;
            border-radius: 6px;
            padding: 12px 16px;
            font-size: 13px;
            color: #003580;
            cursor: pointer;
            text-align: left;
            transition: background 0.2s, transform 0.1s;
            font-family: Arial, sans-serif;
        }
        .ql-btn:hover { background: #ddeeff; transform: translateX(3px); }
        .ql-btn span { margin-right: 8px; }

        .status-list { list-style: none; }
        .status-list li {
            display: flex;
            align-items: center;
            justify-content: space-between;
            padding: 7px 0;
            border-bottom: 1px solid #f0f0f0;
            font-size: 13px;
        }
        .dot { width: 10px; height: 10px; border-radius: 50%; display: inline-block; margin-right: 8px; }
        .dot.green { background: #2ecc71; box-shadow: 0 0 6px #2ecc71; }
        .dot.yellow { background: #f39c12; box-shadow: 0 0 6px #f39c12; }
        .dot.red { background: #e74c3c; box-shadow: 0 0 6px #e74c3c; }

        .contact-item { font-size: 13px; padding: 6px 0; border-bottom: 1px solid #f0f0f0; color: #444; }
        .contact-item strong { color: #002855; }

        .footer { text-align: center; padding: 20px; color: #aaa; font-size: 11px; }

        #toast {
            position: fixed;
            bottom: 30px;
            right: 30px;
            background: #002855;
            color: white;
            padding: 12px 20px;
            border-radius: 8px;
            font-size: 13px;
            opacity: 0;
            transition: opacity 0.3s;
            pointer-events: none;
        }
        #toast.show { opacity: 1; }
    </style>
</head>
<body>

<div class="header">
    <div class="header-left">
        <h1>&#9670; DIGITARK OÜ</h1>
        <p>Siseveeb — ainult töötajatele</p>
    </div>
    <div class="header-right">
        <div id="clock">00:00:00</div>
        <div id="date"></div>
    </div>
</div>

<div class="nav">
    <a class="active">Avaleht</a>
    <a onclick="showToast('Dokumendid — tulemas')">Dokumendid</a>
    <a onclick="showToast('Personal — tulemas')">Personal</a>
    <a onclick="showToast('IT tugi — tulemas')">IT tugi</a>
    <a onclick="showToast('Kontakt — tulemas')">Kontakt</a>
</div>

<div class="container">
    <div>
        <div class="card" style="margin-bottom:20px">
            <h2>Teadaanded</h2>

            <div class="notice" onclick="toggleNotice(this)">
                <div class="notice-title">
                    <span>Serveri hooldus — 15. november</span>
                    <span class="arrow">&#9658;</span>
                </div>
                <div class="notice-date">12. november 2024 — IT osakond</div>
                <div class="notice-body">15. novembril kell 22:00 toimub planeeritud serveri hooldus. Süsteemid on kättesaamatud kuni 2 tundi. Palume tööd aegsasti salvestada.</div>
            </div>

            <div class="notice" onclick="toggleNotice(this)">
                <div class="notice-title">
                    <span>Uus paroolipoliitika jõustub</span>
                    <span class="arrow">&#9658;</span>
                </div>
                <div class="notice-date">1. november 2024 — Juhtkond</div>
                <div class="notice-body">Alates 1. detsembrist kehtib uus paroolipoliitika. Parool peab olema vähemalt 12 tähemärki ning sisaldama suurtähte, numbrit ja erimärki. IT osakond võtab ühendust.</div>
            </div>

            <div class="notice" onclick="toggleNotice(this)">
                <div class="notice-title">
                    <span>Jõulupidu — 20. detsember</span>
                    <span class="arrow">&#9658;</span>
                </div>
                <div class="notice-date">28. oktoober 2024 — Juhtkond</div>
                <div class="notice-body">Digitark OÜ jõulupidu toimub 20. detsembril kell 18:00 restoranis Kolm Õde. Palume registreeruda hiljemalt 10. detsembriks personaliosakonnale.</div>
            </div>
        </div>

        <div class="card">
            <h2>Süsteemi olek</h2>
            <ul class="status-list">
                <li><span><span class="dot green"></span>Active Directory</span> <span style="color:#2ecc71;font-size:12px">Töötab</span></li>
                <li><span><span class="dot green"></span>Meiliserver</span> <span style="color:#2ecc71;font-size:12px">Töötab</span></li>
                <li><span><span class="dot green"></span>Veebiserver (IIS)</span> <span style="color:#2ecc71;font-size:12px">Töötab</span></li>
                <li><span><span class="dot yellow"></span>VPN</span> <span style="color:#f39c12;font-size:12px">Aeglane</span></li>
                <li><span><span class="dot green"></span>Varundus</span> <span style="color:#2ecc71;font-size:12px">Töötab</span></li>
            </ul>
        </div>
    </div>

    <div>
        <div class="card" style="margin-bottom:20px">
            <h2>Kiirlingid</h2>
            <div class="quicklinks">
                <button class="ql-btn" onclick="showToast('Avamine...')"><span>&#128196;</span>Sisedokumendid</button>
                <button class="ql-btn" onclick="showToast('Avamine...')"><span>&#128222;</span>Telefoniraamat</button>
                <button class="ql-btn" onclick="showToast('Avamine...')"><span>&#128197;</span>Puhkusekalender</button>
                <button class="ql-btn" onclick="showToast('Avamine...')"><span>&#128187;</span>IT tugipilet</button>
                <button class="ql-btn" onclick="showToast('Avamine...')"><span>&#128241;</span>Mobiilirakendus</button>
            </div>
        </div>

        <div class="card">
            <h2>Kontakt</h2>
            <div class="contact-item"><strong>IT osakond</strong><br>it@digitark.ee</div>
            <div class="contact-item"><strong>Personal</strong><br>hr@digitark.ee</div>
            <div class="contact-item"><strong>Juhtkond</strong><br>info@digitark.ee</div>
            <div class="contact-item"><strong>Aadress</strong><br>Tartu mnt 12, Tallinn</div>
        </div>
    </div>
</div>

<div class="footer">Digitark OÜ siseveeb &mdash; &copy; 2024 &mdash; Kõik õigused kaitstud</div>

<div id="toast"></div>

<script>
    function updateClock() {
        const now = new Date();
        const h = String(now.getHours()).padStart(2, '0');
        const m = String(now.getMinutes()).padStart(2, '0');
        const s = String(now.getSeconds()).padStart(2, '0');
        document.getElementById('clock').textContent = h + ':' + m + ':' + s;

        const days = ['Pühapäev','Esmaspäev','Teisipäev','Kolmapäev','Neljapäev','Reede','Laupäev'];
        const months = ['jaanuar','veebruar','märts','aprill','mai','juuni','juuli','august','september','oktoober','november','detsember'];
        document.getElementById('date').textContent = days[now.getDay()] + ', ' + now.getDate() + '. ' + months[now.getMonth()] + ' ' + now.getFullYear();
    }
    setInterval(updateClock, 1000);
    updateClock();

    function toggleNotice(el) {
        el.classList.toggle('open');
    }

    let toastTimer;
    function showToast(msg) {
        const t = document.getElementById('toast');
        t.textContent = msg;
        t.classList.add('show');
        clearTimeout(toastTimer);
        toastTimer = setTimeout(() => t.classList.remove('show'), 2500);
    }
</script>

</body>
</html>
```
