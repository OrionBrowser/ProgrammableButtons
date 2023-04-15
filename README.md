# Programmable Buttons

[![](https://dcbadge.vercel.app/api/server/gKh5E6ys6D?compact=true&style=flat)](https://discord.gg/gKh5E6ys6D) [![Twitter](https://img.shields.io/twitter/follow/KagiHQ?style=social)](https://twitter.com/KagiHQ) [![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](https://opensource.org/license/mit/) 

This repository contains open-source code snippets for programmable buttons in Orion browser.

Orion is modern, high performance, WebKit based, zero-telemetry browser for Apple devices.

[Download Orion](https://browser.kagi.com).

## Buttons

Copy the link to the below buttons to import them in Orion. To import right click toolbar and then select "Import Button from URL".

### AI buttons

- [Unbiased News](https://github.com/OrionBrowser/ProgrammableButtons/raw/main/buttons/Unbiased%20News.plist): Uses OpenAI API to produce unbiased rewrite of the news article. Make sure to replace apiKey in the code after importing.
- [Summarize](https://github.com/OrionBrowser/ProgrammableButtons/raw/main/buttons/Summarize.plist): Uses OpenAI API to produce summary of the page. Make sure to replace apiKey in the code after importing.
- [Stock Analysis](https://github.com/OrionBrowser/ProgrammableButtons/raw/main/buttons/Unbiased%20News.plist): Uses OpenAI API to produce stock analysis based on the content of the page. Works best on financial sites like seekingalpha.com. Make sure to replace apiKey in the code after importing.

## Browser interactions

[Dark Mode](https://github.com/OrionBrowser/ProgrammableButtons/raw/main/buttons/Dark%20Mode.plist): Attempts to enable Dark Mode on the page. Uses [this dark mode snippet](https://github.com/OrionBrowser/DarkMode).
[Kill Sticky](https://github.com/OrionBrowser/ProgrammableButtons/raw/main/buttons/Kill%20Sticky.plist): Remove sticky elements and restore scrolling to web pages. Uses [this kill sticky snippet](https://github.com/t-mart/kill-sticky).

## Example 

The following code connects to OpenAI API (change apiKey with [your own](https://platform.openai.com/account/api-keys)) and summarizes the text on the page. It uses streaming support to stream the output directly into Orion sidebar.

Make sure to enable Sidebar button in the button settings.

```
(async () => {
    const apiKey = 'your_api_key_here';
    const text = document.title + ' ' + Array.from(document.querySelectorAll('p')).map(p => p.innerText).join(' ');
    const requestBody = {
        'model': 'gpt-3.5-turbo',
        'messages': [
            {'role': 'system', 'content': 'Summarize this text.'},
            {'role': 'user', 'content': `${text}`}
        ],
        'max_tokens': 800,
        'temperature': 0,
        'stream': true

    };

    const response = await fetch('https://api.openai.com/v1/chat/completions', {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json',
            'Authorization': `Bearer ${apiKey}`
        },
        body: JSON.stringify(requestBody)
    });

    if (response.ok) {
        const reader = response.body.getReader();
        const decoder = new TextDecoder('utf-8');
        let summary = '';

      while (true) {
        const { value, done } = await reader.read();
        if (done) {
            break;
        }
        const lines = decoder.decode(value).split('\n').filter(line => line.trim() !== '');
        for (const line of lines) {
            const message = line.replace(/^data: /, '');
            if (message === '[DONE]') {
                break;
            }
            try {
                const parsed = JSON.parse(message);
                const content = parsed.choices?.[0]?.delta?.content;
                if (content) {
                    summary += content;
                    OrionInternals.setSidebarContent(summary);
                }
            } catch (error) {
                console.error('Could not JSON parse stream message', message, error);
            }
        }
      }

    } else {
        console.error('API request failed:', await response.text());
    }
})();



 ```
