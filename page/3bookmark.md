---
layout: page
title: Bookmark
permalink: /collection/
icon: bookmark
type: page
---

* content
{:toc}

### 2022-2023

<html>
<head>
    <title>GitHub Markdown Viewer</title>
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
                // Display the content in the designated div
                document.getElementById('markdown-content').innerText = decodedContent;
            })
            .catch(error => {
                console.error('Error fetching content:', error);
            });
    </script>
</body>
</html>



## Comments

{% include comments.html %}
