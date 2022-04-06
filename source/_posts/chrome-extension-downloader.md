---
title: Chrome 插件打包下载
date: 2017-12-12 08:12:02
categories: Tips
tags:
	- Chrome
---
到[应用商店](https://chrome.google.com/webstore/category/extensions)搜索并获取获取插件 id
{% asset_img get_id.png %}

<body>
    <input type="text" id="input_id" placeholder="extension id" style="border: 1px solid #cccccc; width: 300px; height: 28px; padding-left: 10px;"><button id="btn" class="btn">生成链接</button>
    <a id="download_link" href="" style="visibility: hidden;">右键另存为</a>
</body>

<script type="text/javascript">
    document.getElementById("btn").onclick = function() {
        var id = document.getElementById("input_id").value;
        if (id != "") {
            var str = "https://clients2.google.com/service/update2/crx?response=redirect&prodversion=49.0&x=id%3D~~%26installsource%3Dondemand%26uc";
            var link = str.replace(/~~/,id);
            document.getElementById("download_link").href = link;
            document.getElementById("download_link").style.visibility = "visible";
		}
		else {
			document.getElementById("download_link").style.visibility = "hidden";
		}
    };
</script>

