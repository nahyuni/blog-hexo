---
title: [avalon nano miner]
date: 2020-02-21 14:30:19
tags: [mining, blockchain, pow]
categories:
    - 프로젝트
thumbnail: ""
permalink: ""
---
 0. ~~ github 소스
 1. avalon-nano 크롬 소스를 분석한다.
 2. 크롬 소스에서 hid SDK(chrome.hid.getDevices)를 nodejs 인 require("node-hid") 로
    컨버팅 한다.(nodejs)
 3. avalon-nano USB 디바이스와 접속 후 통신을 한다.(firmware 통신)
 <!-- more -->
  3.1 Avalon4_USB_HID_Protocol_Guide 문서 분석.
    - Packet 총 길이는 40 Byte
    - Head(2)   | Type(1) | Opt(1) | Idx(1) | Cnt(1) | Data(32) | Crc(2)
        0~3 version(4) 
        4~35 prevhash(32) 
        36~67 merkleRoot(32)
        68~71 ntime(4) 
        72~75 nbits(4)
        76~79 nonce(4)
        
    3.2 P_WORK part 1 : midstat(32) => sha256(input->(0,64)) 
    - version(4) + prevhash(32) + merkleRoot(28) = 64

    3.3 P_WORK part 2: id(6) + data(12) 
    - 32~37 - poolId(1), jqId(1), nonce2(4) -> id(6)
    - 52~63 - merkleRoot(midstat에 나온 마지막 merkleRoot 4바이트), ntime(4), nbits(4) -> 12
 <br>
 4. 검증 작업(80 Byte)
    1. version + prevhash + merkleRoot(32) + ntime + nbits + nonce => 4바이트 리틀 엔디안으로 바꿔서 더블 해쉬로 만듦.(regen_hash)
    

- node-hid 접속
```
r hid = require("node-hid");
var connect = function() {
	dev_con = new hid.HID(VENDOR_ID, ID);
	detect();
};

var listen = function() {
	dev_con.on("data", myListener);
};

function regen_hash(input_str) {
	var hex = SHA256(hex2littlebinary(input_str), 80);
	return SHA256(hex2bigbinary(hex), 32, 0, 1);
}

function hex2littlebinary(str)
{
	var result = new Array();
	for(var i = 0; i < str.length; i += 8) {
		var number = 0x00000000;
		for(var j = 0; j < 4; ++j) {
			number = safe_add(number, hex_to_byte(str.substring(i + j*2, i + j*2 + 2)) << (j*8));
		}
		result.push(number);
	}
	return result;
}

function hex2bigbinary(str)
{
	var result = new Array();
	for(var i = 0; i < str.length; i += 8) {
		var number = 0x00000000;
		for(var j = 0, k = 3; j < 4; ++j, --k) {
			number = safe_add(number, hex_to_byte(str.substring(i + j*2, i + j*2 + 2)) << (k*8));
		}
		result.push(number);
	}
	return result;
}

function hex_to_byte(hex)
{
	return( parseInt(hex, 16));
}

function safe_add (x, y) {
	var lsw = (x & 0xFFFF) + (y & 0xFFFF);
	var msw = (x >> 16) + (y >> 16) + (lsw >> 16);
	return (msw << 16) | (lsw & 0xFFFF);
}	
TEST
```

    