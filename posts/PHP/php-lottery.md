---
title: 大转盘抽奖
date: 2019-02-12 19:17:00
description: PHP 大转盘抽奖
slug: php-lottery
image:
categories:
    - PHP
tags: ["PHP"]

---



```js
// startbtn 开始抽奖按钮
$("#startbtn").click(function(){
        console.log("开始");
        hand.style.webkitAnimation = "";
        $("#startbtn").unbind('click').css("cursor","default");
        $.ajax({
            type: "GET", //提交方式
            url: '/lottery/chou',
            success: function(result) {
                console.log(result);
                if(result.code != 1000) {
                    var con = confirm(result.info);
                    if(con){
                        window.location.href="/lottery/index";
                    }else{
                        $("#windowbg").hide();
                        $("#windowcjBox").hide();
                        return false;
                    }
                }else{
                    var data = result.data;
                    var min = data["rotate_min"]; // 最小角度
                    var max = data["rotate_max"]; // 最大角度

                    console.log(max)
                    console.log(min)
                    //指定范围随机角度
                    var a = Math.floor(min + Math.random() * (max - min));
                    console.log("角度:" + a);
                    var p = "惊喜大礼包"; //奖项
                    // #zhuanBox转盘
                    $("#zhuanBox").rotate({
                        duration:5000, //转动时间
                        angle: 0,
                        animateTo:1800+a, //转动角度
                        easing: $.easing.easeOutSine,
                        callback: function(){
                            // 抽中奖品
                            setTimeout(function () {
                                $("#windowcjBox").hide();
                                // var data = result.data;
                                ds_code = data["coinimg"];
                                qrcode = data["qrcode"];
                                showCode(ds_code); // 弹窗显示抽中奖品信息
                            }, 1000);
                        }
                    });
                }
            }
        });
    });

```



