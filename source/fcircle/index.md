---
title: 朋友圈
date: 2024-08-10 10:55:16
---
<div id="friend-circle-lite-root"></div>
<script>
    if (typeof UserConfig === 'undefined') {
        var UserConfig = {
            // 填写你的fc Lite地址
            private_api_url: 'https://friends-admibrill.pages.dev/',
            // 点击加载更多时，一次最多加载几篇文章，默认20
            page_turning_number: 20,
            // 头像加载失败时，默认头像地址
            error_img: 'https://pic.imgdb.cn/item/6695daa4d9c307b7e953ee3d.jpg',
        }
    }
</script>
<link rel="stylesheet" href="https://fastly.jsdelivr.net/gh/admibrill/Friend-Circle-Lite/main/fclite.min.css">
<script src="https://fastly.jsdelivr.net/gh/admibrill/Friend-Circle-Lite/main/fclite.min.js"></script>
