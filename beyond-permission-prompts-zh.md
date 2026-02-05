# 超越权限提示：让 Claude Code 更安全和自主

Beyond permission prompts: making Claude Code more secure and autonomous---
https://www.anthropic.com/engineering/claude-code-sandboxing

发布日期：2025年10月20日

Published Oct 20, 2025

Claude Code 的新沙箱功能、bash 工具和 Web 版 Claude Code，通过启用文件系统和网络隔离两个边界，减少权限提示并提高用户安全性。

Claude Code's new sandboxing features, a bash tool and Claude Code on the web, reduce permission prompts and increase user safety by enabling two boundaries: filesystem and network isolation.

在 Claude Code 中，Claude 与你一起编写、测试和调试代码，导航你的代码库、编辑多个文件，并运行命令来验证其工作。给予 Claude 如此多访问代码库和文件的权限可能会带来风险，特别是在提示注入的情况下。

In Claude Code, Claude writes, tests, and debugs code alongside you, navigating your codebase, editing multiple files, and running commands to verify its work. Giving Claude this much access to your codebase and files can introduce risks, especially in the case of prompt injection.

为了帮助解决这个问题，我们在 Claude Code 中引入了两个基于沙箱构建的新功能，这两个功能都旨在为开发者提供一个更安全的工作场所，同时也允许 Claude 更自主地运行并减少权限提示。在我们的内部使用中，我们发现沙箱安全地将权限提示减少了 84%。通过定义 Claude 可以自由工作的设定边界，它们提高了安全性和自主性。

To help address this, we've introduced two new features in Claude Code built on top of sandboxing, both of which are designed to provide a more secure place for developers to work, while also allowing Claude to run more autonomously and with fewer permission prompts. In our internal usage, we've found that sandboxing safely reduces permission prompts by 84%. By defining set boundaries within which Claude can work freely, they increase security and agency.

## 保护 Claude Code 用户安全

Keeping users secure on Claude Code

Claude Code 基于权限模型运行：默认情况下，它是只读的，这意味着它会在进行修改或运行任何命令之前请求权限。这有一些例外：我们自动允许安全命令，如 echo 或 cat，但大多数操作仍需要明确批准。

Claude Code runs on a permission-based model: by default, it's read-only, which means it asks for permission before making modifications or running any commands. There are some exceptions to this: we auto-allow safe commands like echo or cat, but most operations still need explicit approval.

不断点击"批准"会减慢开发周期，并可能导致"批准疲劳"，用户可能不会密切关注他们正在批准的内容，进而使开发变得不那么安全。

Constantly clicking "approve" slows down development cycles and can lead to 'approval fatigue', where users might not pay close attention to what they're approving, and in turn making development less safe.

为了解决这个问题，我们为 Claude Code 推出了沙箱功能。

To address this, we launched sandboxing for Claude Code.

## 沙箱：一种更安全和自主的方法

Sandboxing: a safer and more autonomous approach

沙箱创建预定义的边界，Claude 可以在其中更自由地工作，而不是为每个操作请求权限。启用沙箱后，你会获得大幅减少的权限提示和增强的安全性。

Sandboxing creates pre-defined boundaries within which Claude can work more freely, instead of asking for permission for each action. With sandboxing enabled, you get drastically fewer permission prompts and increased safety.

我们的沙箱方法建立在操作系统级功能之上，以实现两个边界：

Our approach to sandboxing is built on top of operating system-level features to enable two boundaries:

**文件系统隔离**，确保 Claude 只能访问或修改特定目录。这对于防止被提示注入的 Claude 修改敏感系统文件尤为重要。

Filesystem isolation, which ensures that Claude can only access or modify specific directories. This is particularly important in preventing a prompt-injected Claude from modifying sensitive system files.

**网络隔离**，确保 Claude 只能连接到已批准的服务器。这可以防止被提示注入的 Claude 泄露敏感信息或下载恶意软件。

Network isolation, which ensures that Claude can only connect to approved servers. This prevents a prompt-injected Claude from leaking sensitive information or downloading malware.

值得注意的是，有效的沙箱需要文件系统隔离和网络隔离两者。没有网络隔离，被入侵的智能体可能会外泄 SSH 密钥等敏感文件；没有文件系统隔离，被入侵的智能体可以轻松逃离沙箱并获得网络访问权限。正是通过使用这两种技术，我们才能为 Claude Code 用户提供更安全、更快速的智能体体验。

It is worth noting that effective sandboxing requires both filesystem and network isolation. Without network isolation, a compromised agent could exfiltrate sensitive files like SSH keys; without filesystem isolation, a compromised agent could easily escape the sandbox and gain network access. It's by using both techniques that we can provide a safer and faster agentic experience for Claude Code users.

## Claude Code 中的两个新沙箱功能

Two new sandboxing features in Claude Code

### 沙箱 bash 工具：无需权限提示的安全 bash 执行

Sandboxed bash tool: safe bash execution without permission prompts

我们推出了一个新的沙箱运行时，作为研究预览版以测试版形式提供，它允许你精确定义智能体可以访问哪些目录和网络主机，而无需启动和管理容器的开销。这可用于沙箱任意进程、智能体和 MCP 服务器。它也作为开源研究预览版提供。

We're introducing a new sandbox runtime, available in beta as a research preview, that lets you define exactly which directories and network hosts your agent can access, without the overhead of spinning up and managing a container. This can be used to sandbox arbitrary processes, agents and MCP servers. It is also available as an open source research preview.

在 Claude Code 中，我们使用这个运行时来沙箱 bash 工具，它允许 Claude 在你设置的定义限制内运行命令。在安全沙箱内，Claude 可以更自主地运行，并安全执行命令而无需权限提示。如果 Claude 尝试访问沙箱外的内容，你会立即收到通知，并可以选择是否允许。

In Claude Code, we use this runtime to sandbox the bash tool, which allows Claude to run commands within the defined limits you set. Inside the safe sandbox, Claude can run more autonomously and safely execute commands without permission prompts. If Claude tries to access something outside of the sandbox, you'll be notified immediately, and can choose whether or not to allow it.

我们在操作系统级原语之上构建了这个功能，例如 Linux bubblewrap 和 MacOS seatbelt，以在操作系统级别强制执行这些限制。它们不仅涵盖 Claude Code 的直接交互，还涵盖命令生成的任何脚本、程序或子进程。如上所述，这个沙箱强制执行：

We've built this on top of OS level primitives such as Linux bubblewrap and MacOS seatbelt to enforce these restrictions at the OS level. They cover not just Claude Code's direct interactions, but also any scripts, programs, or subprocesses that are spawned by the command.As described above, this sandbox enforces both:

**文件系统隔离**，允许对当前工作目录进行读写访问，但阻止修改其外部的任何文件。

Filesystem isolation, by allowing read and write access to the current working directory, but blocking the modification of any files outside of it.

**网络隔离**，只允许通过连接到沙箱外运行的代理服务器的 unix 域套接字访问互联网。这个代理服务器强制执行进程可以连接的域的限制，并处理新请求域的用户确认。如果你想要进一步提高安全性，我们还支持自定义此代理以对出站流量强制执行任意规则。

Network isolation, by only allowing internet access through a unix domain socket connected to a proxy server running outside the sandbox. This proxy server enforces restrictions on the domains that a process can connect to, and handles user confirmation for newly requested domains. And if you'd like further-increased security, we also support customizing this proxy to enforce arbitrary rules on outgoing traffic.

这两个组件都是可配置的：你可以轻松选择允许或禁止特定的文件路径或域。

Both components are configurable: you can easily choose to allow or disallow specific file paths or domains.

**此图说明了 Claude Code 中沙箱的工作原理。**

This image illustrations how sandboxing in Claude Code works.

Claude Code 的沙箱架构通过文件系统和网络控制隔离代码执行，自动允许安全操作，阻止恶意操作，并仅在需要时请求权限。

Claude Code's sandboxing architecture isolates code execution with filesystem and network controls, automatically allowing safe operations, blocking malicious ones, and asking permission only when needed.

沙箱确保即使提示注入成功，也完全隔离，不会影响整体用户安全性。这样，被入侵的 Claude Code 无法窃取你的 SSH 密钥，或向攻击者的服务器发送信息。

Sandboxing ensures that even a successful prompt injection is fully isolated, and cannot impact overall user security. This way, a compromised Claude Code can't steal your SSH keys, or phone home to an attacker's server.

要开始使用此功能，请在 Claude Code 中运行 `/sandbox` 并查看有关我们安全模型的更多技术细节。

To get started with this feature, run /sandbox in Claude Code and check out more technical details about our security model.

为了让其他团队更容易构建更安全的智能体，我们已将此功能开源。我们相信其他人应该考虑为他们自己的智能体采用这项技术，以增强其智能体的安全态势。

To make it easier for other teams to build safer agents, we have open sourced this feature. We believe that others should consider adopting this technology for their own agents in order to enhance the security posture of their agents.

### Web 版 Claude Code：在云端安全运行 Claude Code

Claude Code on the web: running Claude Code securely in the cloud

今天，我们还发布了 Web 版 Claude Code，使用户能够在云端的隔离沙箱中运行 Claude Code。Web 版 Claude Code 在隔离沙箱中执行每个 Claude Code 会话，它可以安全可靠地完全访问其服务器。我们设计这个沙箱是为了确保敏感凭证（如 git 凭证或签名密钥）永远不会与 Claude Code 一起在沙箱内。这样，即使沙箱中运行的代码被入侵，用户也能免受进一步伤害。

Today, we're also releasing Claude Code on the web enabling users to run Claude Code in an isolated sandbox in the cloud. Claude Code on the web executes each Claude Code session in an isolated sandbox where it has full access to its server in a safe and secure way. We've designed this sandbox to ensure that sensitive credentials (such as git credentials or signing keys) are never inside the sandbox with Claude Code. This way, even if the code running in the sandbox is compromised, the user is kept safe from further harm.

Web 版 Claude Code 使用自定义代理服务，透明地处理所有 git 交互。在沙箱内，git 客户端使用自定义构建的范围凭证向此服务进行身份验证。代理验证此凭证和 git 交互的内容（例如，确保仅推送到配置的分支），然后在将请求发送到 GitHub 之前附加正确的身份验证令牌。

Claude Code on the web uses a custom proxy service that transparently handles all git interactions. Inside the sandbox, the git client authenticates to this service with a custom-built scoped credential. The proxy verifies this credential and the contents of the git interaction (e.g. ensuring it is only pushing to the configured branch), then attaches the right authentication token before sending the request to GitHub.

**此图描述了 Web 版 Claude Code 如何使用自定义代理处理所有 git 交互。**

This illustration depicts how Claude Code on the web uses a custom proxy to handle all git interactions.

Claude Code 的 Git 集成通过安全代理路由命令，该代理验证身份验证令牌、分支名称和仓库目标——允许安全的版本控制工作流，同时防止未经授权的推送。

Claude Code's Git integration routes commands through a secure proxy that validates authentication tokens, branch names, and repository destinations—allowing safe version control workflows while preventing unauthorized pushes.

## 开始使用

Getting started

我们新的沙箱 bash 工具和 Web 版 Claude Code 为使用 Claude 进行工程工作的开发者在安全性和生产力方面都提供了实质性改进。

Our new sandboxed bash tool and Claude Code on the web offer substantial improvements in both security and productivity for developers using Claude for their engineering work.

要开始使用这些工具：

To get started with these tools:

在 Claude 中运行 `/sandbox` 并查看我们的文档了解如何配置此沙箱。

Run `/sandbox` in Claude and check out our docs on how to configure this sandbox.

访问 claude.com/code 试用 Web 版 Claude Code。

Go to claude.com/code to try out Claude Code on the web.

或者，如果你正在构建自己的智能体，请查看我们的开源沙箱代码，并考虑将其集成到你的工作中。我们期待看到你构建的内容。

Or, if you're building your own agents, check out our open-sourced sandboxing code, and consider integrating it into your work. We look forward to seeing what you build.

要了解更多关于 Web 版 Claude Code 的信息，请查看我们的发布博客文章。

To learn more about Claude Code on the web, check out our launch blog post.

## 致谢

Acknowledgements

本文由 David Dworken 和 Oliver Weller-Davies 撰写，Meaghan Choi、Catherine Wu、Molly Vorwerck、Alex Isken、Kier Bradwell 和 Kevin Garcia 做出了贡献。

Article written by David Dworken and Oliver Weller-Davies, with contributions from Meaghan Choi, Catherine Wu, Molly Vorwerck, Alex Isken, Kier Bradwell, and Kevin Garcia
