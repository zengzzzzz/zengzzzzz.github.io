---
layout: page
title: Bookmark
permalink: /collection/
icon: bookmark
type: page
---

* content
{:toc}

### 2023-05-07 - now

<html>
<head>
    <title>GitHub Markdown Viewer</title>
    <script src="https://cdn.jsdelivr.net/npm/marked/marked.min.js"></script>
</head>
<body>
    <div id="markdown-content"></div>

    <script>
        // GitHub content URL from your server
        const githubContentUrl = 'https://raw.githubusercontent.com/zengzzzzz/the-reading-history/main/README.md';

        // Fetch the content from the custom server
        fetch(githubContentUrl)
            .then(response => response.text())
            .then(decodedContent => {
                // Use marked.js to convert Markdown to HTML
                const renderedHtml = marked(decodedContent);
                document.getElementById('markdown-content').innerHTML = renderedHtml;
            })
            .catch(error => {
                console.error('Error fetching content:', error);
            });
    </script>
</body>
</html>


## Comments

{% include comments.html %}
