# Programmable Buttons

[![](https://dcbadge.vercel.app/api/server/gKh5E6ys6D?compact=true&style=flat)](https://discord.gg/gKh5E6ys6D) [![Twitter](https://img.shields.io/twitter/follow/KagiHQ?style=social)](https://twitter.com/KagiHQ) [![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](https://opensource.org/license/mit/) 

This repository contains open-source code snippets for programmable buttons in Orion browser.

Orion is modern, high performance, WebKit based, zero-telemetry browser for Apple devices.

[Download Orion browser by Kagi](https://browser.kagi.com).

## Programmable Button demos

Usage:
- Right click and copy the link to the button. 
- Right click Orion toolbar and select "Import Button from URL". 

### Browser interaction buttons

- [Dark Mode](https://github.com/OrionBrowser/ProgrammableButtons/raw/main/buttons/Dark%20Mode.plist): Attempts to enable Dark Mode on the page. Uses Orion's [dark mode snippet](https://github.com/OrionBrowser/DarkMode).
- [Kill Sticky](https://github.com/OrionBrowser/ProgrammableButtons/raw/main/buttons/Kill%20Sticky.plist): Remove sticky elements and restore scrolling to web pages. Uses this [kill sticky snippet](https://github.com/t-mart/kill-sticky).
- [Translate with Google](https://github.com/OrionBrowser/ProgrammableButtons/raw/main/buttons/Translate%20with%20Google.plist): Re-loads the page in Google Translate for automatic translation.
- [Modern CSS](https://github.com/OrionBrowser/ProgrammableButtons/raw/main/buttons/Modern%20CSS.plist): Applies modern CSS to the page. Useful for pages with no styles. Test [here](https://danluu.com/futurist-predictions/). Uses [Water.css stylesheet](https://watercss.kognise.dev/).
- [Classic CSS](https://github.com/OrionBrowser/ProgrammableButtons/raw/main/buttons/Classic%20CSS.plist): Applies classic web era CSS to the page. Useful for pages with no styles. Test [here](https://danluu.com/futurist-predictions/). Uses ['old style' W3C stylesheet](https://www.w3.org/StyleSheets/Core/preview).

### AI buttons

You will need to input your OpenAI API key in the code for these to work. When you import the button, right click it, select edit, switch to code, and then replace OpenAI API key on the top of the code.

- [Unbiased News](https://github.com/OrionBrowser/ProgrammableButtons/raw/main/buttons/Unbiased%20News.plist): Uses OpenAI API to produce unbiased rewrite of the news article. Make sure to replace apiKey in the code after importing.
- [Summarize](https://github.com/OrionBrowser/ProgrammableButtons/raw/main/buttons/Summarize.plist): Uses OpenAI API to produce summary of the page. Make sure to replace apiKey in the code after importing. This is just a proof of concept and will not work with all pages. For a more robust summarization API, consider using [Universal Summarizer](https://kagi.com/summarizer). 
- [Stock Analysis](https://github.com/OrionBrowser/ProgrammableButtons/raw/main/buttons/Unbiased%20News.plist): Uses OpenAI API to produce stock analysis based on the content of the page. Works best on financial sites like seekingalpha.com. Make sure to replace apiKey in the code after importing.

## Community Buttons
Community members are welcome to contribute their own programmable buttons in the [Community Buttons folder](https://github.com/OrionBrowser/ProgrammableButtons/tree/main/community_buttons).

Currently available are:
- [Copy page link as Markdown](https://github.com/OrionBrowser/ProgrammableButtons/blob/main/community_buttons/Copy%20page%20link%20as%20Markdown.plist): Copies the page link as a Markdown link.
- [Discuss document or page](https://github.com/OrionBrowser/ProgrammableButtons/blob/main/community_buttons/Discuss%20doc%20or%20page.plist): Ask questions or discuss a document or page via AI chat.

## Demo

Cick the thumbnail to watch the video.

[![Watch the video](https://img.youtube.com/vi/xoJliN5Pwv8/hqdefault.jpg)](https://www.youtube.com/watch?v=xoJliN5Pwv8)

## Pro tip

You can inspect the code, icons and options used in the button by converting the file into a more readable format like xml.

For example:
```
plutil -convert xml1 -o Summarize.xml Summarize.plist
```

## Example 

### External API call

The code will call an external API and display result in Orion sidebar.

```
    (async () => {
        const apiKey = 'your_api_key';
        const text = document.title + ' ' + Array.from(document.querySelectorAll('p')).map(p => p.innerText).join(' ');
        const requestBody = {
            'model': 'gpt-3.5-turbo',
            'messages': [
                { 'role': 'system', 'content': 'Your task is to summarise the text I have given you in up to seven concise bullet points, starting with a short highlight. \nYour output should use the following template: \nSummary \nHighlights \n-  Bulletpoint \n"' },
                { 'role': 'user', 'content': `${text}` }
            ],
            'max_tokens': 800,
            'temperature': 0
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
            const data = await response.json();
            const summary = data.choices[0].message.content;
            OrionInternals.setSidebarContent(summary)
        } else {
            console.error('API request failed:', await response.text());
        }
    })();
 ```

### Streaming response from OpenAI API
The following is a demonstration of the code used to power the 'Summarize' button.

The code connects to OpenAI API (change apiKey with [your own](https://platform.openai.com/account/api-keys)) and sends a prompt that will summarizes the text on the page. It uses OrionInternals.setSidebarContent(summary)  method to update Orions sidebar.


```
(async () => {
    const apiKey = 'your_api_key';
    const text = document.title + ' ' + Array.from(document.querySelectorAll('p')).map(p => p.innerText).join(' ');
    const requestBody = {
        'model': 'gpt-3.5-turbo',
        'messages': [
            { 'role': 'system', 'content': 'Your task is to summarise the text I have given you in up to seven concise bullet points, starting with a short highlight. \nYour output should use the following template: \nSummary \nHighlights \n-  Bulletpoint \n"' },
            { 'role': 'user', 'content': `${text}` }
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
                    const content = parsed.choices && parsed.choices[0] && parsed.choices[0].delta && parsed.choices[0].delta.content;
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
