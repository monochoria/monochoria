---
layout: post
title: js + node.js 实现简单的生成列表音乐播放器
category: coding
---

专业课（数字媒体资源管理）的作业 02 ，要求做一个根据文件夹内存放的音乐自动生成歌曲信息列表的播放器，使用 Python + Flask 或者 JS + Node.js 。不太熟 Python 所以入门了一下 Node.js 就开始写了，结果非常弱鸡但好歹能用……继续学 Node.js + AJAX + jQuery 之后打算重写一遍。也想学学 vue.js ……

思路就是 Node.js 用 cheerio （很不优雅地）操作 DOM 写表，然后 JS 从 DOM 的表中存取信息。

extractID3.js
```
// node.js 部分，使用了 cherrio 操作 DOM , jsmediatags 抽取歌曲 ID3 

const cheerio = require("cheerio");
const jsmediatags = require("jsmediatags");
const glob = require("glob");
const path = require("path");
const fs = require("fs");

// todo: JSON Array

var musicNumber = new Array();
var musicTitle = new Array();
var musicImage = new Array();
var musicArtist = new Array();
var musicAlbum = new Array();

function getID3(){
    glob.sync( '../MP3/*.mp3' ).forEach( function( file ) {
        jsmediatags.read(file, {
            onSuccess: function(tag) {
            var tags = tag.tags;

            // Gen Image

            var pic = tags.picture.data;
            var buffer = new Buffer(pic.length);
            for(var i = 0; i < pic.length; i++)
                buffer[i] = pic[i];
            musicImage.push(buffer);


            // Gen info

            musicNumber.push(file);
            musicTitle.push(tags.title);
            musicArtist.push(tags.artist);
            musicAlbum.push(tags.album);

            }
        });
    });

}

// Use cheerio to write DOM inelegantly

// todo: use AJAX

function writeTable(){
    var content = fs.readFileSync("../player.html");
    $ = cheerio.load(content);

    $('#toWrite').empty();
    for(var i=0;i<200;i++){
        $('#toWrite').append("<tr><th scope='row'><button type='button' class='btn btn-default' aria-label='play' value='"+musicNumber[i].slice(7,-4)+"' onclick='getValue(this.value)'><span class='glyphicon glyphicon-play' aria-hidden='true'></span></button></th>"+"<td>"+musicNumber[i].slice(7,-4)+"</td>"+"<td>"+musicTitle[i]+"</td>"+"<td>"+musicArtist[i]+"</td>"+"<td>"+musicAlbum[i]+"</td></tr>");
    }
    fs.writeFile("../player.html", $.html(), function(err){
        if(err)
            throw err;
        //console.log("write in table");

    });
}

// todo: learn Events

setTimeout(function(){
    writeTable();
}, 8000);

getID3();
```

main.js
```
// Gloabl vars

var playing = -1;

// play buttons

function setValue(val){
  if(val == 1 && playing != "200"){ // prev song

    playing++;
  }
  else if(val == -1 && playing != "001"){ // next song

    playing--;
  }
  else if(val == 0){ // random song

    // Avoid from gen song 0

    do{playing = parseInt(Math.random()*200);}while(playing==0);
  }
  else{ // song 0 or song 200

    alert("Error: over songID");
    return;
  }
  // Deal with ID

  var playStr = playing.toString();
  if(playStr.length == 1)
    playStr = "00"+playStr;
  else if(playStr.length == 2)
    playStr = "0"+playStr;
  else;
    getValue(playStr);
  getValue(playStr);
}

// Deal with got value from button

function getValue(val){
  playing = val;
  if(playing == -1 || playing[0] != "0");
  else{
    if(playing[0] == "0" && playing[1] == "0")
      val = val[2];
    else{
      val = val.slice(1,3);
    }
  }
  //console.log(val);

  // show info

  var table = document.getElementById("toWrite");
  document.getElementById("musicPlayer").src="./MP3/"+playing+".mp3";
  document.getElementById("img-info").src="./image/"+playing+".png";
  
  /*
   * When using node.js to write in DOM,
   * sometimes the list will be shuffled (I don't know why),
   * so it is need to find the correct row for the selected song.
  */
  for(var i=0;i<table.rows.length;i++){
    if(table.rows[i].cells[1].innerText==playing){
      document.getElementById("title-info").innerText = table.rows[i].cells[2].innerText;
      document.getElementById("artist-info").innerText = table.rows[i].cells[3].innerText;
      document.getElementById("album-info").innerText = table.rows[i].cells[4].innerText;
      break;
    }
  }
  // todo: use jQuery: document.ready()
}
```