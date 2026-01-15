# vk-band-battle
<!DOCTYPE html><html lang="ja">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>V系バトルジェネレーター</title>
  <style>
    body {
      font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;
      background: linear-gradient(180deg, #0b0b0b, #1a1a1a);
      color: #fff;
      margin: 0;
      padding: 16px;
    }
    h1 { text-align: center; color: #7ecbff; font-size: 1.4rem; }
    .box { background:#111; border-radius:16px; padding:16px; margin-bottom:16px; }
    input { width:100%; font-size:16px; padding:12px; border-radius:12px; border:none; margin-bottom:8px; }
    button { width:100%; padding:14px; font-size:18px; border-radius:20px; border:none; background:linear-gradient(90deg,#5fa9ff,#9b6bff); color:#fff; font-weight:bold; margin-bottom:16px; }
    .log { background:#000; border-radius:16px; padding:16px; white-space:pre-line; }
    ul { padding-left:18px; }
    .admin { background:#300; margin-top:8px; padding:8px; border-radius:8px; }
    .del { color:#ff7777; cursor:pointer; margin-left:8px; }
  </style>
</head>
<body>
<h1>🎸 ヴィジュアル系バンド<br>戦闘力バトル 🎸</h1><div class="box">
  <h2>🖤 あなたのバンド</h2>
  <input id="player" placeholder="バンド名を入力" />
</div>
<div class="box">
  <h2>💀 対戦バンド</h2>
  <input id="cpu" placeholder="CPUバンド名を入力" />
</div>
<button onclick="battle()">🔥 バトル開始 🔥</button><div class="box log" id="result">ここに結果が表示される…</div><div class="box">
  <h2>🏆 戦闘力ランキング（上位5）</h2>
  <ul id="ranking"></ul>
</div><div class="box">
  <h2>🔥 連勝ランキング（上位5）</h2>
  <ul id="streakRank"></ul>
</div><div class="box admin">
  <h2>🔑 管理人モード</h2>
  <input id="adminPass" placeholder="管理人パスワード" />
  <button onclick="adminLogin()">管理人ログイン</button>
</div><script>
  const ADMIN_PASSWORD = 'admin123'; // ★ここを変更してOK
  let isAdmin = false;

  function calcPower(name) {
    let power = 0;
    for (let i = 0; i < name.length; i++) power += name.charCodeAt(i);
    if (name.match(/[゛ー]/)) power += 500;
    return power % 10000 + 1000;
  }

  function load(key) {
    return JSON.parse(localStorage.getItem(key) || '[]');
  }
  function save(key, val) {
    localStorage.setItem(key, JSON.stringify(val));
  }

  function adminLogin() {
    if (adminPass.value === ADMIN_PASSWORD) {
      isAdmin = true;
      alert('管理人モードON');
      renderRanking();
      renderStreakRanking();
    } else {
      alert('パスワードが違います');
    }
  }

  function renderRanking() {
    const ul = document.getElementById('ranking');
    ul.innerHTML = '';
    load('powerRank').forEach((r,i)=>{
      const li = document.createElement('li');
      li.innerHTML = `${i+1}位：${r.band} / ${r.user}（${r.power}）` +
        (isAdmin ? `<span class='del' onclick="delPower(${i})">[削除]</span>` : '');
      ul.appendChild(li);
    });
  }

  function renderStreakRanking() {
    const ul = document.getElementById('streakRank');
    ul.innerHTML = '';
    load('streakRank').forEach((r,i)=>{
      const li = document.createElement('li');
      li.innerHTML = `${i+1}位：${r.band} / ${r.user}（${r.streak}連勝）` +
        (isAdmin ? `<span class='del' onclick="delStreak(${i})">[削除]</span>` : '');
      ul.appendChild(li);
    });
  }

  function delPower(i) {
    const list = load('powerRank');
    list.splice(i,1);
    save('powerRank', list);
    renderRanking();
  }
  function delStreak(i) {
    const list = load('streakRank');
    list.splice(i,1);
    save('streakRank', list);
    renderStreakRanking();
  }

  function battle() {
    const p = player.value.trim();
    const c = cpu.value.trim();
    if (!p || !c) return;

    const pPower = calcPower(p);
    const cPower = calcPower(c);

    let log = `🖤 ${p}\n戦闘力：${pPower}\n\n💀 ${c}\n戦闘力：${cPower}\n\n`;
    let streak = Number(localStorage.getItem('streak') || 0);

    if (p === c) {
      log += '⚠️ 同名バンドのため記録対象外';
    } else if (pPower > cPower) {
      log += '🔥 勝利！';
      streak++;
      localStorage.setItem('streak', streak);
      checkStreakRank(p, streak);
    } else if (pPower < cPower) {
      log += '💀 敗北…';
      localStorage.setItem('streak', 0);
    } else {
      log += '⚡ 引き分け';
    }

    document.getElementById('result').textContent = log;
    checkPowerRank(p, pPower);
  }

  function checkPowerRank(band, power) {
    let list = load('powerRank');
    if (list.length < 5 || power > list[list.length-1].power) {
      const user = prompt('ランキング入り！ユーザーネームを入力');
      if (!user) return;
      list.push({ band, user, power });
      list.sort((a,b)=>b.power-a.power);
      save('powerRank', list.slice(0,5));
      renderRanking();
    }
  }

  function checkStreakRank(band, streak) {
    let list = load('streakRank');
    if (list.length < 5 || streak > list[list.length-1].streak) {
      const user = prompt('連勝ランキング入り！ユーザーネームを入力');
      if (!user) return;
      list.push({ band, user, streak });
      list.sort((a,b)=>b.streak-a.streak);
      save('streakRank', list.slice(0,5));
      renderStreakRanking();
    }
  }

  renderRanking();
  renderStreakRanking();
</script></body>
</html>
