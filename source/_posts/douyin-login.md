---
title: 模拟登陆
categories:
  - 抖音
excerpt: ''
description: ''
date: 2024-05-14 09:07:03
---

> 代码参考：https://github.com/Superheroff/douyin_uplod

## 依赖文件

requirements.txt
```
greenlet==3.0.3
playwright==1.43.0
pyee==11.1.0
typing_extensions==4.11.0
```

## 代码实现

```python
# -*- coding: utf-8 -*-

import asyncio
import os
from playwright.async_api import Playwright, async_playwright


class creator_douyin():
    def __init__(self, phone, timeout: int):
        """
        初始化
        :param phone: 手机号
        :param timeout: 你要等待多久，单位秒
        """
        self.timeout = timeout * 1000
        self.phone = phone
        self.path = os.path.abspath('')
        self.desc = "cookie_%s.json" % phone

        if not os.path.exists(os.path.join(self.path, "cookie")):
            os.makedirs(os.path.join(self.path, "cookie"))

    async def __cookie(self, playwright: Playwright) -> None:
        browser = await playwright.chromium.launch(channel="chrome", headless=False)

        context = await browser.new_context()

        page = await context.new_page()
        await page.add_init_script("Object.defineProperties(navigator, {webdriver:{get:()=>undefined}});")

        await page.goto("https://creator.douyin.com/")

        await page.locator(
            "div.banner-div:nth-child(1) > div:nth-child(1) > img:nth-child(1)").click()

        
        # # 手机号登陆
        # await page.locator("#semiTabphone").click()
        # await page.locator('#number').fill(self.phone)
        # await page.locator(".agreement-yykwUw > img:nth-child(1)").click()
        try:
            await page.wait_for_url("https://creator.douyin.com/creator-micro/home", timeout=self.timeout)
            cookies = await context.cookies()
            cookie_txt = ''
            for i in cookies:
                cookie_txt += i.get('name') + '=' + i.get('value') + '; '

            with open(os.path.join(self.path, "cookie", self.phone + ".txt"), mode="w") as f:
                f.write(cookie_txt)

            await context.storage_state(path=os.path.join(self.path, "cookie", self.desc))
            print(self.phone + " ——> 登录成功")
        except Exception as e:
            print(self.phone + " ——> 登录失败，本次操作不保存cookie", e)
        finally:
            await page.close()
            await context.close()
            await browser.close()

    async def main(self):
        async with async_playwright() as playwright:
            await self.__cookie(playwright)


def main():
    while True:
        phone = input('请输入手机号码\n输入"exit"将退出服务\n')
        if phone == "exit":
            break
        elif phone.isnumeric() and len(phone) == 11:
            app = creator_douyin(phone, 60)
            asyncio.run(app.main())
        else:
            print('请输入正确的手机号码\n')

main()
```
