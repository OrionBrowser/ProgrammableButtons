# Programmable Buttons

This repository contains open-source code snippets for programmable buttons in Orion browser.

Orion is modern, high performance, WebKit based, zero-telemetry browser for Apple devices.

[Download Orion](https://browser.kagi.com).

## Unbiased News

```
(async () => {
    const apiKey = 'your_api_key_here';
    const text = document.title + ' ' + Array.from(document.querySelectorAll('p')).map(p => p.innerText).join(' ');
    const requestBody = {
        'model': 'gpt-3.5-turbo',
        'messages': [
            {'role': 'system', 'content': '- Rewrite the  article to make it informative, truth-focused, and neutral.\n- Using the heading "Biases", describe the biases (with examples) of the original article (in bullet points)\n- Using the heading  "Other issues",  describe (with examples) the logical contradictions and fallacies in the original (in bullet points)\n- Rate the journalism in the original article with a "Journalism score" between 1 -10 based on the quality of journalism . 1/10 means extremely biased journalism with clear agenda and 10/10 is informative, objective journalism. '},
            {'role': 'user', 'content': `${text}`}
        ],
        'max_tokens': 800,
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
