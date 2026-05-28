

# Your Final Stack

| Purpose                | Tool                                                       |
| ---------------------- | ---------------------------------------------------------- |
| Writing notes          | [Obsidian](https://obsidian.md/?utm_source=chatgpt.com)    |
| Notes format           | Markdown (`.md`)                                           |
| Convert to website     | [Quartz](https://quartz.jzhao.xyz/?utm_source=chatgpt.com) |
| Hosting                | [Netlify](https://www.netlify.com/?utm_source=chatgpt.com) |
| Backup/version control | [GitHub](https://github.com/?utm_source=chatgpt.com)       |
run locally
`npx quartz build --serve`

**Push Your Site**
`npx quartz sync --no-pull`

This commits your content and pushes everything to your repository. For subsequent updates, just run:

`npx quartz sync`