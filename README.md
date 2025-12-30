<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Chess</title>

<style>
body{
  font-family:sans-serif;
  text-align:center;
}
table{
  border-collapse:collapse;
  margin:auto;
}
td{
  width:12vw;
  height:12vw;
  max-width:70px;
  max-height:70px;
  font-size:8vw;
  text-align:center;
  touch-action:manipulation;
}
.w{ background:white; }
.b{ background:skyblue; }
.sel{ outline:4px solid red; }
</style>
</head>

<body>

<h2 id="status">White to move</h2>
<table id="board"></table>

<script>
const W="♙♖♘♗♕♔", B="♟♜♞♝♛♚";
let turn="white", sel=null;

let board=[
              ["♜","♞","♝","♛","♚","♝","♞","♜"],
              ["♟","♟","♟","♟","♟","♟","♟","♟"],
              ["","","","","","","",""],
              ["","","","","","","",""],
              ["","","","","","","",""],
              ["","","","","","","",""],
              ["♙","♙","♙","♙","♙","♙","♙","♙"],
              ["♖","♘","♗","♕","♔","♗","♘","♖"]
];

const el=document.getElementById("board");
const status=document.getElementById("status");

const isW=p=>W.includes(p);
const isB=p=>B.includes(p);

function draw(){
  el.innerHTML="";
  for(let r=0;r<8;r++){
    let tr=document.createElement("tr");
    for(let c=0;c<8;c++){
      let td=document.createElement("td");
      td.className=(r+c)%2?"b":"w";
      td.textContent=board[r][c];
      td.onclick=()=>tap(r,c,td);
      tr.appendChild(td);
    }
    el.appendChild(tr);
  }
}

function tap(r,c,td){
  let p=board[r][c];
  if(sel){
    if(legal(sel.r,sel.c,r,c)){
      let save=board[r][c];
      board[r][c]=board[sel.r][sel.c];
      board[sel.r][sel.c]="";

      if(inCheck(turn)){
        board[sel.r][sel.c]=board[r][c];
        board[r][c]=save;
      }else{
        turn=turn==="white"?"black":"white";
        if(inCheck(turn) && checkmate(turn)){
          status.textContent="CHECKMATE! "+(turn==="white"?"Black":"White")+" wins";
        }else if(inCheck(turn)){
          status.textContent=(turn==="white"?"White":"Black")+" is in CHECK";
        }else{
          status.textContent=(turn==="white"?"White":"Black")+" to move";
        }
      }
    }
    sel=null;
    draw();
  }else if(p && ((turn==="white"&&isW(p))||(turn==="black"&&isB(p)))){
    sel={r,c};
    td.classList.add("sel");
  }
}

function legal(sr,sc,er,ec){
  let p=board[sr][sc], t=board[er][ec];
  if(t && isW(p)===isW(t)) return false;
  let dr=er-sr, dc=ec-sc;

  if(p==="♙"){
    if(sc===ec && dr===-1 && !t) return true;
    if(sr===6 && sc===ec && dr===-2 && !t) return true;
    if(Math.abs(dc)===1 && dr===-1 && isB(t)) return true;
  }

  if(p==="♟"){
    if(sc===ec && dr===1 && !t) return true;
    if(sr===1 && sc===ec && dr===2 && !t) return true;
    if(Math.abs(dc)===1 && dr===1 && isW(t)) return true;
  }

  if("♖♜".includes(p) && (sr===er||sc===ec)) return clear(sr,sc,er,ec);
  if("♗♝".includes(p) && Math.abs(dr)===Math.abs(dc)) return clear(sr,sc,er,ec);
  if("♕♛".includes(p) &&
    (sr===er||sc===ec||Math.abs(dr)===Math.abs(dc))) return clear(sr,sc,er,ec);

  if("♘♞".includes(p) &&
    ((Math.abs(dr)===2&&Math.abs(dc)===1)||(Math.abs(dr)===1&&Math.abs(dc)===2))) return true;

  if("♔♚".includes(p) && Math.abs(dr)<=1 && Math.abs(dc)<=1) return true;

  return false;
}

function clear(sr,sc,er,ec){
  let dr=Math.sign(er-sr), dc=Math.sign(ec-sc);
  for(let r=sr+dr,c=sc+dc;r!==er||c!==ec;r+=dr,c+=dc)
    if(board[r][c]) return false;
  return true;
}

function inCheck(color){
  let kr,kc;
  for(let r=0;r<8;r++)for(let c=0;c<8;c++){
    if(board[r][c]===(color==="white"?"♔":"♚")){kr=r;kc=c;}
  }
  for(let r=0;r<8;r++)for(let c=0;c<8;c++){
    let p=board[r][c];
    if(p && ((color==="white"&&isB(p))||(color==="black"&&isW(p))))
      if(legal(r,c,kr,kc)) return true;
  }
  return false;
}

function checkmate(color){
  for(let r=0;r<8;r++)for(let c=0;c<8;c++){
    let p=board[r][c];
    if(p && ((color==="white"&&isW(p))||(color==="black"&&isB(p)))){
      for(let r2=0;r2<8;r2++)for(let c2=0;c2<8;c2++){
        if(legal(r,c,r2,c2)){
          let a=board[r2][c2], b=board[r][c];
          board[r2][c2]=b; board[r][c]="";
          let chk=inCheck(color);
          board[r][c]=b; board[r2][c2]=a;
          if(!chk) return false;
        }
      }
    }
  }
  return true;
}

draw();
</script>

</body>
</html>
