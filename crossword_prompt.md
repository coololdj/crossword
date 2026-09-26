【指示】添付画像のクロスワードパズルを読み取り、スマホ向け単一HTMLアプリを作成してください、
【必須仕様】
①盤面構成: 画像から7x7マスの黒マス・数字位置・カギ文章を正確に解析して配置すること。
認識後の黒の数を各行列で足し算して、合否判定し合うまでやり直すこと。
②入力・自動分解: ローマ字・ひらがな・カタカナ入力を「全角カタカナ」に変換し、複数文字も1文字ずつ分解して進行方向の白マスへ連続展開すること。
③操作性: 入力後の自動フォーカス移動、マス再タップ、鍵文による「ヨコ ⇄ タテ」自動方向切り替えを実装すること。
④不要機能除外: キーボード、ヒント、正解判定、エフェクト等は含めないこと。

以下は見本です。データーは置き換えること。追記は不可。
<!doctype html>
<html lang="ja">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>7×7 クロスワード</title>
<style>
*{box-sizing:border-box}body{margin:0;padding:12px;background:#f6f7f9;color:#26313d;font-family:-apple-system,BlinkMacSystemFont,"Segoe UI","Hiragino Sans","Yu Gothic UI",sans-serif;-webkit-tap-highlight-color:transparent}main{max-width:390px;margin:auto}header{display:flex;align-items:center;justify-content:space-between;margin:0 2px 10px}h1{margin:0;font-size:1.18rem}.tools{display:flex;gap:7px}.dir,.clear,.tab{border:0;border-radius:7px;font-weight:700}.dir{padding:5px 9px;color:#fff;background:#d94686}.dir.down{background:#3b82f6}.clear{padding:6px 9px;background:#e9edf2;color:#485565}.board{display:grid;grid-template-columns:repeat(7,1fr);gap:1px;padding:2px;background:#253244;border-radius:5px;aspect-ratio:1}.cell{position:relative;display:flex;align-items:center;justify-content:center;min-width:0;background:#fff;font:700 clamp(1.18rem,6vw,1.55rem)/1 sans-serif;color:#17202b;user-select:none}.cell.black{background:#253244}.cell.active{background:#fde68a}.cell.word{background:#fef3c7}.num{position:absolute;top:2px;left:3px;font-size:.57rem;font-weight:700;color:#566273}.cluebar{display:flex;gap:8px;align-items:flex-start;margin:10px 0;padding:9px 10px;border:1px solid #cddff7;border-radius:8px;background:#eff6ff;font-size:.88rem;line-height:1.4}.badge{flex:none;padding:2px 7px;border-radius:5px;background:#d94686;color:#fff;font-size:.72rem;font-weight:700}.badge.down{background:#3b82f6}.panel{overflow:hidden;border:1px solid #d8dee7;border-radius:9px;background:#fff}.tabs{display:grid;grid-template-columns:1fr 1fr;background:#f1f4f7}.tab{padding:9px;background:transparent;color:#617083;border-radius:0}.tab.on{background:#fff;color:#17202b}.list{max-height:220px;overflow:auto;padding:5px}.clue{padding:7px 8px;border-radius:6px;font-size:.86rem;line-height:1.45;cursor:pointer}.clue.active{background:#e8f1ff;font-weight:600}.clue b{margin-right:5px;color:#d94686}.downlist .clue b{color:#3b82f6}.empty{padding:14px 8px;text-align:center;color:#8a96a5;font-size:.82rem}.hint{margin:8px 2px 0;color:#738091;font-size:.72rem;line-height:1.5}#ime{position:fixed;left:-20px;bottom:0;width:1px;height:1px;opacity:.01;border:0;padding:0;font-size:16px}
</style>
</head>
<body>
<main>
<header><h1>7×7 クロスワード</h1><div class="tools"><button id="dir" class="dir">ヨコ</button><button id="clear" class="clear">クリア</button></div></header>
<div id="board" class="board"></div>
<div class="cluebar"><span id="badge" class="badge">ヨコ</span><span id="current">マスをタップして入力してください</span></div>
<section class="panel">
<div class="tabs"><button class="tab on" data-d="across">ヨコのカギ</button><button class="tab" data-d="down">タテのカギ</button></div>
<div id="list" class="list"></div>
</section>
<p class="hint">日本語IMEで確定した文字だけを、選択中の語へ1文字ずつ配置します。</p>
</main>
<textarea id="ime" lang="ja" autocomplete="off" autocorrect="off" autocapitalize="off" spellcheck="false"></textarea>

<script>
/* ===== 画像読取りデータ部：ここだけ置換 =====
   GRID : "."=白マス  "#"=黒マス
   CLUES: 盤面番号:{ text:"カギ本文" }
*/
const GRID=[
".......",
".......",
".......",
".......",
".......",
".......",
"......."
];
const CLUES={across:{},down:{}};
/* ===== データ部ここまで ===== */

const N=7,$=s=>document.getElementById(s),board=$("board"),list=$("list"),
dirBtn=$("dir"),badge=$("badge"),current=$("current"),ime=$("ime");
let nums={},pos={},r=0,c=0,dir="across",tab="across",composing=false,skip=false,
V=Array.from({length:N},()=>Array(N).fill(""));

for(let y=0,n=0;y<N;y++)for(let x=0;x<N;x++)if(GRID[y][x]!="#"){
 let a=(x==0||GRID[y][x-1]=="#")&&x+1<N&&GRID[y][x+1]!="#",
 d=(y==0||GRID[y-1][x]=="#")&&y+1<N&&GRID[y+1][x]!="#";
 if(a||d){nums[y+"-"+x]=++n;pos[n]={r:y,c:x}}
}

function cells(y,x,d){
 let z=[],dy=d=="down",dx=!dy,Y=y,X=x;
 while(Y-dy>=0&&X-dx>=0&&GRID[Y-dy][X-dx]!="#"){Y-=dy;X-=dx}
 while(Y<N&&X<N&&GRID[Y][X]!="#"){z.push({r:Y,c:X});Y+=dy;X+=dx}
 return z
}
function clue(y,x,d){
 let w=cells(y,x,d),p=w[0],n=nums[p.r+"-"+p.c],q=CLUES[d][n];
 return w.length>1&&q?{n,text:q.text,cells:w}:null
}
function choose(y,x){
 if(clue(y,x,dir))return;
 let d=dir=="across"?"down":"across";
 if(clue(y,x,d))dir=d
}
function focusCell(y,x){
 if(y<0||y>=N||x<0||x>=N||GRID[y][x]=="#")return;
 r=y;c=x;choose(r,c);paint();ime.value="";ime.focus({preventScroll:true})
}
function toggle(){
 let d=dir=="across"?"down":"across";
 if(cells(r,c,d).length>1){dir=d;paint();ime.focus({preventScroll:true})}
}
function paint(){
 document.querySelectorAll(".cell").forEach(e=>e.classList.remove("active","word"));
 let q=clue(r,c,dir),w=cells(r,c,dir);
 if(w.length>1)w.forEach(p=>$(`c-${p.r}-${p.c}`).classList.add(p.r==r&&p.c==c?"active":"word"));
 else $(`c-${r}-${c}`)?.classList.add("active");
 if(q){
  badge.textContent=(dir=="across"?"ヨコ":"タテ")+q.n;
  badge.className="badge"+(dir=="down"?" down":"");
  current.textContent=q.text
 }else{
  let p=w[0],n=p&&nums[p.r+"-"+p.c];
  badge.textContent=(dir=="across"?"ヨコ":"タテ")+(n||"");
  badge.className="badge"+(dir=="down"?" down":"");
  current.textContent=n?"カギ未設定":"カギなし"
 }
 dirBtn.textContent=dir=="across"?"ヨコ":"タテ";
 dirBtn.className="dir"+(dir=="down"?" down":"");
 renderList()
}
function kana(s){
 return [...s.normalize("NFKC").replace(/[ぁ-ゖ]/g,ch=>String.fromCharCode(ch.charCodeAt(0)+96))]
 .filter(ch=>/[ァ-ヺー]/.test(ch))
}
function setChar(y,x,ch){V[y][x]=ch;$(`c-${y}-${x}`).lastChild.nodeValue=ch}
function commit(s){
 let a=kana(s);ime.value="";if(!a.length)return;
 let w=cells(r,c,dir);if(w.length<2){setChar(r,c,a[0]);return}
 let k=w.findIndex(p=>p.r==r&&p.c==c),last=k;
 for(let i=0;i<a.length&&k+i<w.length;i++){let p=w[k+i];setChar(p.r,p.c,a[i]);last=k+i}
 let nx=w[Math.min(last+1,w.length-1)];r=nx.r;c=nx.c;paint();ime.focus({preventScroll:true})
}
function back(){
 if(V[r][c]){setChar(r,c,"");return}
 let w=cells(r,c,dir),k=w.findIndex(p=>p.r==r&&p.c==c);
 if(k>0){let p=w[k-1];setChar(p.r,p.c,"");focusCell(p.r,p.c)}
}
function move(dy,dx){focusCell(r+dy,c+dx)}
function renderList(){
 list.className="list"+(tab=="down"?" downlist":"");list.innerHTML="";
 let entries=Object.entries(CLUES[tab]);
 if(!entries.length){let e=document.createElement("div");e.className="empty";e.textContent="カギ未設定";list.append(e);return}
 entries.forEach(([n,q])=>{
  let e=document.createElement("div"),p=pos[n],a=clue(r,c,dir);e.className="clue";
  if(tab==dir&&a&&+n==a.n)e.classList.add("active");
  e.innerHTML=`<b>${n}.</b>${q.text}`;
  if(p)e.onclick=()=>{dir=tab;focusCell(p.r,p.c)};
  list.append(e)
 })
}

for(let y=0;y<N;y++)for(let x=0;x<N;x++){
 let d=document.createElement("div");d.id=`c-${y}-${x}`;d.className="cell";
 if(GRID[y][x]=="#")d.classList.add("black");
 else{
  let n=nums[y+"-"+x];if(n){let s=document.createElement("span");s.className="num";s.textContent=n;d.append(s)}
  d.append(document.createTextNode(""));
  d.onclick=()=>r==y&&c==x?toggle():focusCell(y,x)
 }
 board.append(d)
}

ime.addEventListener("compositionstart",()=>composing=true);
ime.addEventListener("compositionend",e=>{composing=false;skip=true;commit(e.data||ime.value);queueMicrotask(()=>skip=false)});
ime.addEventListener("input",()=>{if(!composing&&!skip&&ime.value)commit(ime.value)});
ime.addEventListener("keydown",e=>{
 if(e.key=="Backspace"&&!composing){e.preventDefault();back()}
 else if(e.key=="ArrowRight"){e.preventDefault();move(0,1)}
 else if(e.key=="ArrowLeft"){e.preventDefault();move(0,-1)}
 else if(e.key=="ArrowDown"){e.preventDefault();move(1,0)}
 else if(e.key=="ArrowUp"){e.preventDefault();move(-1,0)}
});

document.querySelectorAll(".tab").forEach(b=>b.onclick=()=>{
 tab=b.dataset.d;
 document.querySelectorAll(".tab").forEach(x=>x.classList.toggle("on",x==b));
 renderList()
});
dirBtn.onclick=toggle;
$("clear").onclick=()=>{
 if(confirm("盤面の文字をすべて消去しますか？")){
  V.forEach(row=>row.fill(""));
  document.querySelectorAll(".cell:not(.black)").forEach(e=>e.lastChild.nodeValue="");
  focusCell(r,c)
 }
};
focusCell(0,0);
</script>
</body>
</html>
