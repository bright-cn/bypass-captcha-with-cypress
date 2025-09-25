# 使用 Cypress 绕过 CAPTCHA

[![Promo](https://github.com/bright-cn/LinkedIn-Scraper/raw/main/Proxies%20and%20scrapers%20GitHub%20bonus%20banner.png)](https://www.bright.cn/)

本指南介绍如何在 Cypress 中处理 CAPTCHA，包括有效的绕过方法，以及当 CAPTCHA 仍然出现时的应对措施，帮助实现流畅的浏览器自动化。

- [什么是 CAPTCHA？能否自动化？](#什么是-captcha能否自动化)
- [CAPTCHA 与 Cypress：一段“艰难”的关系](#captcha-与-cypress一段艰难的关系)
- [如何在 Cypress 中处理 CAPTCHA](#如何在-cypress-中处理-captcha)
  - [方法一：禁用 CAPTCHA](#方法一禁用-captcha)
  - [方法二：自动化与 CAPTCHA 的交互](#方法二自动化与-captcha-的交互)
  - [方法三：集成反检测浏览器](#方法三集成反检测浏览器)
- [这些 Cypress 方案都不奏效时该怎么办？](#这些-cypress-方案都不奏效时该怎么办)

## 什么是 CAPTCHA？能否自动化？

CAPTCHA（全称为 Completely Automated Public Turing test to tell Computers and Humans Apart）是一种安全机制，用于区分真实用户与自动化脚本。它会给出对人类较易、对机器较难的挑战，常被放置在网页关键位置以阻止机器行为。

### 常见 CAPTCHA 类型

常见的提供商包括 Google reCAPTCHA、hCaptcha 和 BotDetect。CAPTCHA 通常属于以下一种或多种类型：

- 文本型 CAPTCHA：要求用户输入一串扭曲的字母或数字。
- 图片型 CAPTCHA：要求用户在图片网格中识别特定物体。
- 音频型 CAPTCHA：要求用户根据音频片段输入听到的词语。
- 拼图型 CAPTCHA：要求用户完成简单挑战，如点击特定对象或回答问题。

![Puzzle CAPTCHA example](https://github.com/bright-cn/bypass-captcha-with-cypress/blob/main/images/Puzzle-CAPTCHA-example-1.png)

### CAPTCHA 的使用方式

CAPTCHA 常被集成在关键用户流程（如表单提交）中，以防止自动化完成操作：

![CAPTCHA in form submission process](https://github.com/bright-cn/bypass-captcha-with-cypress/blob/main/images/CAPTCHA-as-a-step-of-a-form-submission-process-example.png)

在这类场景中，CAPTCHA 始终显示，无法通过简单的自动化技巧绕过。一些解题服务会使用人工操作员或专门的 AI 模型实时解题，但硬编码强制显示 CAPTCHA 的情况相对较少，因为它会影响用户体验。

### 高级反机器人系统中的 CAPTCHA

更常见的是，CAPTCHA 是高级反机器人系统（如 Web 应用防火墙 WAF）（[了解更多](https://www.cloudflare.com/learning/ddos/glossary/web-application-firewall-waf/)）的一部分：

![Web Application Firewall example](https://github.com/bright-cn/bypass-captcha-with-cypress/blob/main/images/Example-of-a-Web-Application-Firewall-1024x488.png)

在此类系统中，网站在怀疑用户可能是机器人时会动态触发 CAPTCHA。这意味着通过让自动化行为更像真人（如使用真实浏览器并模拟人类交互），有时可以避免触发。

然而，绕过 CAPTCHA 始终充满挑战，因为反爬与反机器人技术在不断演进。自动化脚本也需要频繁更新，以对抗新的检测手段。

## CAPTCHA 与 Cypress：一段“艰难”的关系

[Cypress](https://www.cypress.io/) 是一款为现代 Web 设计的前端测试工具。尽管它也可用于网页自动化（如抓取），但其核心定位是[端到端（E2E）测试](https://www.techtarget.com/searchsoftwarequality/definition/End-to-end-testing)。这意味着 Cypress 最适合在您可控的网站与应用上使用。

当 Cypress 用于外部或第三方网站时，可能会出现问题。[官方文档](https://docs.cypress.io/guides/references/best-practices)建议尽量避免与第三方站点交互。关键原因之一是存在被识别为机器人并遇到 CAPTCHA 的风险。

CAPTCHA 的设计目标就是阻止自动化脚本，包括在 Cypress 测试环境中运行的脚本。一旦出现 CAPTCHA，可能会打断 Cypress 的自动化流程，导致测试无法顺利完成。

不过，尽管在 Cypress 中绕过 CAPTCHA 具有挑战性，但并非完全不可能。接下来将介绍在 Cypress 自动化中处理 CAPTCHA 的潜在方案与权衡。

## 如何在 Cypress 中处理 CAPTCHA

虽然 CAPTCHA 是 Cypress 的主要挑战之一，但也不必就此放弃。让我们探讨几种可行思路。

### 方法一：禁用 CAPTCHA

大多数 CAPTCHA 提供商允许在测试环境中禁用或绕过挑战。如果您能够控制目标站点，考虑在测试环境中完全关闭 CAPTCHA，或替换成更简单的版本。

例如，使用 reCAPTCHA v3 时，您可以为测试环境创建单独的密钥。对于 reCAPTCHA v2，可使用以下测试密钥：

- Site key：`6LeIxAcTAAAAAJcZVRqyHh71UMIEGNQ_MXjiZKhI`
- Secret key：`6LeIxAcTAAAAAGG-vFI1TnRWxMZNFuojJ4WifJWe`

使用这些密钥时，您将总是看到如下的 reCAPTCHA “No CAPTCHA” 小组件：

![](https://github.com/bright-cn/bypass-captcha-with-cypress/blob/main/images/recaptcha_test.png)

该设置会显示特殊警告信息，说明此 CAPTCHA 不适用于生产环境。自动点击此提示即可确保反机器人验证总是通过。更多详情参见 [reCAPTCHA 文档](https://developers.google.com/recaptcha/docs/faq#id-like-to-run-automated-tests-with-recaptcha.-what-should-i-do)。

其他 CAPTCHA 提供商也为测试环境提供类似选项。

### 方法二：自动化与 CAPTCHA 的交互

有些 CAPTCHA 只需要基础交互，例如点击一个复选框（如 reCAPTCHA 的 “No CAPTCHA” 组件）。

![Simple clicking CAPTCHA example](https://github.com/bright-cn/bypass-captcha-with-cypress/blob/main/images/Simple-clicking-CAPTCHA-example.png)

虽然这类挑战看似简单，但往往会分析鼠标移动等行为信号来判断是否为真人。不过，并非所有 CAPTCHA 都如此先进；有些仅用于阻挡最基础的脚本，因而更易绕过。在这类情况下，可以尝试用 Cypress 的逻辑进行自动化。

检查上例的 CAPTCHA 元素会发现它嵌入在一个 iframe 中。

![Inspecting the CAPTCHA element](https://github.com/bright-cn/bypass-captcha-with-cypress/blob/main/images/Inspecting-the-CAPTCHA-element-1024x510.png)

这对大多数 CAPTCHA 提供商来说都很常见。

由于 Cypress 无法自动处理跨域 iframe，请在 `cypress.json` 文件中[将 `chromeWebSecurity` 设置为 `false`](https://docs.cypress.io/guides/guides/web-security#Set-chromeWebSecurity-to-false)：

```json
{

chromeWebSecurity: false

}
```

随后，选择 CAPTCHA 的复选框元素并点击。对于 reCAPTCHA 的 “No CAPTCHA” 组件，示例自动化代码如下：

```js
cy.get('iframe[src*=recaptcha]')

.its('0.contentDocument')

.should(d => d.getElementById('recaptcha-token').click())
```

需要注意的是，CAPTCHA 已足够智能，能区分机器人点击与人类点击，因此此变通方法在大多数情况下并不可靠。今日可行的方案，明日可能失效。欲了解最新做法，可查看该方案来源的 [GitHub gist](https://gist.github.com/Dema/dcc2b4f91b97ec9d56afc871e66fd3d7)。

### 方法三：集成反检测浏览器

前两种绕过方法依赖假设较多。更高效的方案是配置 Cypress 去控制[反检测浏览器](https://www.bright.cn/blog/proxy-101/best-antidetect-browsers)。

默认情况下，Cypress 可使用本地已安装的以下浏览器之一（见[列表](https://docs.cypress.io/guides/guides/launching-browsers#Browsers)）：

- Chrome
- Chrome Beta
- Chrome Canary
- Chromium
- Edge
- Edge Beta
- Edge Canary
- Edge Dev
- Electron
- Firefox
- Firefox Developer Edition
- Firefox Nightly
- WebKit（试验性）

它也支持任意基于 Chromium 的浏览器。请选择其中一个基于 Chromium 的浏览器并安装。

然后，指示 Cypress 使用指定浏览器启动脚本：

```bash
cypress open --browser <path_to_your_browser>
```

其中 `<path_to_your_browser>` 为包含您的反检测浏览器可执行文件的文件夹绝对路径。

同样，您可以在 `cypress.config.js` 中添加如下代码，让 Cypress UI 将该反检测浏览器显示为可选择项：

```js
import { defineConfig } from 'cypress'

export default defineConfig({

e2e: {

setupNodeEvents(on, config) {

const antidetectBrowser = {

name: '<ANTIDETECT_BROWSER_NAME>',

channel: 'stable',

family: 'chromium',

displayName: '<ANTIDETECT_BROWSER_DISPLAY_NAME>',

version,

path: '<path_to_your_browser>',

majorVersion,

}

return {

browsers: config.browsers.concat(antidetectBrowser),

}

},

},

})
```

> 注意：
>
> 让 Cypress 在反检测浏览器中运行自动化代码只能降低被识别为机器人的概率。如果反机器人系统识别出您在运行自动化，它仍可能强制触发 CAPTCHA 来阻断。

## 这些 Cypress 方案都不奏效时该怎么办？

上述三种方法各有明显局限：

- 方法一：要求您能访问目标站点代码，对外部在线站点不适用。
- 方法二：仅对非常简单的 CAPTCHA 有效，且可靠性不足。
- 方法三：意味着额外成本，只是降低触发 CAPTCHA 的概率，不能真正“解题”。

尽管都值得尝试，但它们都无法让您在 Cypress 自动化中以编程方式稳定地绕过 CAPTCHA。

如果您需要真正的 Cypress CAPTCHA 处理方案，试试 Bright Data 的网页抓取解决方案。其具备[专用 CAPTCHA 解决功能](https://www.bright.cn/products/web-unlocker/captcha-solver)，可自动处理 reCAPTCHA、hCaptcha、px_captcha、SimpleCaptcha、GeeTest CAPTCHA、FunCaptcha、Cloudflare Turnstile、AWS WAF Captcha、KeyCAPTCHA 等多种类型。

现在即可开启免费试用。
