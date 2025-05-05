
Below is a concise SOP (Standard Operating Procedure) for applying the Zettelkasten (卡片盒) 笔记法，分为五个主要步骤，每步都配备具体操作及注意事项，帮助你在 Obsidian 或任何笔记工具中高效落地。

> **概览**：Zettelkasten 核心在于“捕捉—整理—连接—复习”四部曲。首先快速捕捉“闪念”；随后针对文献或素材生成文献笔记；再将有价值的思考转写为永久笔记；通过索引笔记巧妙组织；最后定期复盘、维护网络结构，以保证笔记体系活跃而有用 ([Getting Started • Zettelkasten Method](https://zettelkasten.de/overview/?utm_source=chatgpt.com), [Concepts of "How to Take Smart Notes" by Sönke Ahrens](https://zettelkasten.de/posts/concepts-sohnke-ahrens-explained/?utm_source=chatgpt.com))。

---

## 准备工作

1. **选定工具与插件**
    
    - 使用 Obsidian 或其它支持双向链接、YAML Frontmatter、Dataview 的笔记软件 ([Getting Started • Zettelkasten Method](https://zettelkasten.de/overview/?utm_source=chatgpt.com))。
        
    - 安装核心插件：Dataview、Templater/QuickAdd、Daily Notes、Obsidian Git（备份）等 ([A Beginner's Guide to the Zettelkasten Method | Zenkit](https://zenkit.com/en/blog/a-beginners-guide-to-the-zettelkasten-method/?utm_source=chatgpt.com))。
        
2. **建立文件夹结构（可选）**
    
    - 建议按照 “闪念/文献/永久/索引” 四大目录分文件夹，或采用 PARA、Zettel 编号体系 ([Different Kinds of Ties Between Notes - Zettelkasten.de](https://zettelkasten.de/posts/kinds-of-ties/?utm_source=chatgpt.com))。
        
3. **定义 Frontmatter 模板**
    
    - 为每种笔记类型创建对应的 YAML 模板，确保字段（如 `type`、`created`、`tags`、`related` 等）统一 ([The Zettelkasten Method: Examples to help you get started. - Medium](https://medium.com/%40fairylights_io/the-zettelkasten-method-examples-to-help-you-get-started-8f8a44fa9ae6?utm_source=chatgpt.com), [My Step by Step Process to Create a Permanent Note in Zettelkasten](https://brookewrites.medium.com/my-step-by-step-process-to-create-a-permanent-note-in-zettelkasten-a4fd356ec636?utm_source=chatgpt.com))。
        

---

## 1. 捕捉闪念（Fleeting Notes）

### 操作流程

1. **即时记录**：在灵感一闪时，立刻打开快捷面板（手机应用、快捷键）记录短句或关键词。 ([Concepts of "How to Take Smart Notes" by Sönke Ahrens](https://zettelkasten.de/posts/concepts-sohnke-ahrens-explained/?utm_source=chatgpt.com))
    
2. **简洁即可**：只需捕捉要点，无需组织结构；留给后续处理。 ([How I Quickly Capture Ideas Using Fleeting Notes and Obsidian MD](https://medium.com/%40johnmavrick/how-i-quickly-capture-ideas-using-fleeting-notes-and-obsidian-md-86c234ed24c1?utm_source=chatgpt.com))
    
3. **设置属性**：Frontmatter 中 `type: Fleeting`，并自动打上 `created`、`id`、`archived: false` 等字段。 ([Concepts of "How to Take Smart Notes" by Sönke Ahrens](https://zettelkasten.de/posts/concepts-sohnke-ahrens-explained/?utm_source=chatgpt.com), [How I Quickly Capture Ideas Using Fleeting Notes and Obsidian MD](https://medium.com/%40johnmavrick/how-i-quickly-capture-ideas-using-fleeting-notes-and-obsidian-md-86c234ed24c1?utm_source=chatgpt.com))
    

### 注意事项

- 不要在此阶段深度思考或整理，避免打断灵感流动。
    
- 建议每日或每周清理：将 `processed: false` 的闪念进行分类，转化为文献或永久笔记，并更新 `processed: true`。 ([Help a newbie: difference between literature notes and permanent ...](https://www.reddit.com/r/Zettelkasten/comments/yf1e8j/help_a_newbie_difference_between_literature_notes/?utm_source=chatgpt.com))
    

---

## 2. 生成文献笔记（Literature Notes）

### 操作流程

1. **阅读笔记**：对书籍、论文或文章做摘录，并用自己的话写下核心观点。 ([The Zettelkasten Method: Examples to help you get started. - Medium](https://medium.com/%40fairylights_io/the-zettelkasten-method-examples-to-help-you-get-started-8f8a44fa9ae6?utm_source=chatgpt.com))
    
2. **Frontmatter 填写**：`type: Literature`，并补充 `title`、`authors`、`year`、`source`、`reading_status` 等字段。 ([r/Zettelkasten on Reddit: Need Help with documenting Step-by-Step ...](https://www.reddit.com/r/Zettelkasten/comments/1cz23c8/need_help_with_documenting_stepbystep_techniques/?utm_source=chatgpt.com))
    
3. **编号与链接**：给每条文献笔记一个唯一 ID（如 `202504301045-lit-Doe2020`），并在模板末尾预留 `notes_page` 指向永久笔记。 ([The Zettelkasten Method: Examples to help you get started. - Medium](https://medium.com/%40fairylights_io/the-zettelkasten-method-examples-to-help-you-get-started-8f8a44fa9ae6?utm_source=chatgpt.com), [My Step by Step Process to Create a Permanent Note in Zettelkasten](https://brookewrites.medium.com/my-step-by-step-process-to-create-a-permanent-note-in-zettelkasten-a4fd356ec636?utm_source=chatgpt.com))
    

### 注意事项

- 保持“一条文献笔记只记录一个文献”原则，方便后续追溯和引用。
    
- 摘录务必用自己的理解重述，避免大段引用。 ([Taking effective notes: the Zettelkasten method - Gianmarco David](https://gianmarcodavid.com/posts/zettelkasten/?utm_source=chatgpt.com))
    

---

## 3. 撰写永久笔记（Permanent/Evergreen Notes）

### 操作流程

1. **激发新思考**：在阅读文献笔记或复习闪念时，挑选能激发你思考的点，写成自主见解。 ([Help a newbie: difference between literature notes and permanent ...](https://www.reddit.com/r/Zettelkasten/comments/yf1e8j/help_a_newbie_difference_between_literature_notes/?utm_source=chatgpt.com))
    
2. **完整句子**：用完整、清晰的句子表达，确保十年后仍能自洽理解。 ([The Zettelkasten Method: Examples to help you get started. - Medium](https://medium.com/%40fairylights_io/the-zettelkasten-method-examples-to-help-you-get-started-8f8a44fa9ae6?utm_source=chatgpt.com))
    
3. **Frontmatter 填写**：`type: Evergreen`，并加入 `title`、`tags`、`related_concepts`、`updated`、`sources` 等字段。 ([My Step by Step Process to Create a Permanent Note in Zettelkasten](https://brookewrites.medium.com/my-step-by-step-process-to-create-a-permanent-note-in-zettelkasten-a4fd356ec636?utm_source=chatgpt.com))
    
4. **链接出发**：在笔记正文中通过 `[[...]]` 将相关概念或其他永久笔记互联。 ([Different Kinds of Ties Between Notes - Zettelkasten.de](https://zettelkasten.de/posts/kinds-of-ties/?utm_source=chatgpt.com))
    

### 注意事项

- 每个笔记只聚焦一个概念或思想。
    
- 避免抄录原文，务必以个人视角重述。 ([My Step by Step Process to Create a Permanent Note in Zettelkasten](https://brookewrites.medium.com/my-step-by-step-process-to-create-a-permanent-note-in-zettelkasten-a4fd356ec636?utm_source=chatgpt.com))
    

---

## 4. 索引与组织（Index/MOC）

### 操作流程

1. **创建索引笔记**：建一个或多个主题索引（MOC），在 Frontmatter 中 `type: Index` 并用 `sections` 列出分块。 ([Introduction to the Zettelkasten Method](https://zettelkasten.de/introduction/?utm_source=chatgpt.com))
    
2. **维护目录结构**：在索引笔记中，通过列表或 Dataview 动态表格聚合各类笔记链接。
    
    ```dataview
    TABLE file.link AS 笔记, tags
    WHERE contains(tags, "Evergreen")
    ```
    
3. **双向链接**：在永久或项目笔记中引用索引笔记，使导航更快捷。 ([Strategies for connecting notes : r/Zettelkasten - Reddit](https://www.reddit.com/r/Zettelkasten/comments/xkrdkk/strategies_for_connecting_notes/?utm_source=chatgpt.com))
    

### 注意事项

- 索引笔记不承载深度内容，只作“地图”使用。
    
- 定期更新，将新笔记纳入相应索引。 ([Strategies for connecting notes : r/Zettelkasten - Reddit](https://www.reddit.com/r/Zettelkasten/comments/xkrdkk/strategies_for_connecting_notes/?utm_source=chatgpt.com))
    

---

## 5. 定期复习与维护

### 操作流程

1. **周期复盘**：每周/每月查看 `processed: false` 或 `reading_status != done` 的笔记，推进流程。 ([How do you capture fleeting notes in your workflow? : r/Zettelkasten](https://www.reddit.com/r/Zettelkasten/comments/te80rv/how_do_you_capture_fleeting_notes_in_your_workflow/?utm_source=chatgpt.com))
    
2. **更新笔记**：为已有永久笔记补充 `updated` 字段，修正或新增关联。 ([My Step by Step Process to Create a Permanent Note in Zettelkasten](https://brookewrites.medium.com/my-step-by-step-process-to-create-a-permanent-note-in-zettelkasten-a4fd356ec636?utm_source=chatgpt.com))
    
3. **归档历史**：对长期不再活跃的闪念或文献笔记，设置 `archived: true`，并移动至 Archive 文件夹。 ([The fleeting note in "How to Take Smart Notes" by Sönke Ahrens](https://forum.zettelkasten.de/discussion/2119/the-fleeting-note-in-how-to-take-smart-notes-by-soenke-ahrens?utm_source=chatgpt.com))
    

### 注意事项

- 保持笔记网络活跃，防止孤立。
    
- 避免笔记过度堆积，随时归档或删除无价值条目。 ([The fleeting note in "How to Take Smart Notes" by Sönke Ahrens](https://forum.zettelkasten.de/discussion/2119/the-fleeting-note-in-how-to-take-smart-notes-by-soenke-ahrens?utm_source=chatgpt.com))
    

---

以上即为完整的 Zettelkasten 卡片盒笔记法 SOP，从捕捉到复盘，建立了闭环流程。按照这一流程，你可以在 Java 开发、技术研究或任何知识工作中持续扩展并高效运用你的第二大脑。