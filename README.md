# 📅 GitHub Daily Trending Report

> 每日 GitHub 热门项目追踪，支持按日期切换查看

---

## 📆 日期选择

<script>
function loadDate(date) {
  document.getElementById('date-title').textContent = date;
  fetch('data/' + date + '.json')
    .then(r => r.json())
    .then(data => render(data))
    .catch(() => renderEmpty());
}

function render(data) {
  let html = '';
  ['daily','weekly','monthly'].forEach(key => {
    const label = {daily:'🕐 今日热榜', weekly:'📆 本周热榜', monthly:'📆 本月热榜'}[key];
    const starLabel = {daily:'⭐ 今日', weekly:'⭐ 本周', monthly:'⭐ 本月'}[key];
    if(!data[key]) return;
    html += `<h3>${label}</h3>
    <table>
    <thead><tr><th>#</th><th>仓库</th><th>${starLabel}</th><th>语言</th><th>描述</th></tr></thead>
    <tbody>`;
    data[key].forEach((r,i) => {
      html += `<tr>
        <td>${i+1}</td>
        <td><a href="https://github.com/${r.repo}" target="_blank">${r.repo}</a></td>
        <td>${r.stars}</td>
        <td>${r.lang}</td>
        <td>${r.use}</td>
      </tr>`;
    });
    html += '</tbody></table>';
  });
  document.getElementById('content').innerHTML = html;
}

function renderEmpty() {
  document.getElementById('content').innerHTML = '<p>暂无数据</p>';
}
</script>

<div id="date-picker"></div>
<h2 id="date-title">选择日期</h2>
<div id="content"><p>请从上方选择日期</p></div>

<script>
// Build date picker
const picker = document.getElementById('date-picker');
const dates = DATA_LIST || [];
dates.slice(-14).reverse().forEach(d => {
  const btn = document.createElement('button');
  btn.textContent = d;
  btn.onclick = () => loadDate(d);
  picker.appendChild(btn);
});
if(dates.length) loadDate(dates[dates.length-1]);
</script>

---

*由 QClaw 自动生成推送*
