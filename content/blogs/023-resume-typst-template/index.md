+++
title = "Streamlined Resume Design Using Typst"
description = "A modern approach to document formatting and styling"
date = 2024-07-04T00:00:00+00:00

[taxonomies]
tags = ["resume", "dsl", "typst"]

[extra]
toc = true
+++

![Image Source — [https://gist.github.com/fenjalien/1463a19ba2b91d061ed35e295494e0b3](https://gist.github.com/fenjalien/1463a19ba2b91d061ed35e295494e0b3)](https://cdn-images-1.medium.com/max/3840/0*MLncUCESSOhwCTOC.png)

In this blog, we will look at how to create beautiful resumes using Typst. Typst is a new markup-based typesetting system for generating documents designed to be a better alternative to advanced tools like [LaTeX](https://www.latex-project.org/).

For years I have been using [latex templates](https://github.com/madhank93/latex-resume-template) to create resumes. I also had my share of problems when using latex, [out of frustration](https://typst.app/about/), two grads from Berlin created this amazing tool called [Typst](https://typst.app/). It gives you familiar programming constructs, a consistent styling system, understandable error messages, and highly readable & maintainable code.

### Typst installation

Typst can be installed through different package managers or get the pre-built binaries from [GitHub](https://github.com/typst/typst/releases). For more details refer [here](https://github.com/typst/typst?tab=readme-ov-file#installation). Along with this a highly recommended VS code plugin [Typst LSP](https://marketplace.visualstudio.com/items?itemName=nvarner.typst-lsp).

### Building a reusable resume template

i) **Setting Global Styles:** Defining a global font color variable.

```js
#let font_color = rgb("#28282B")
#let font_color_headings = rgb("#1B1212")
```
ii) **Utility Functions:**

* This function helps align two blocks of text side by side.

```js
#let justify_align(left_body, right_body) = {
    block[
    #left_body
    #box(width: 1fr)[#align(right)[#right_body]]
    ]
}
```

* This function generates a section title with consistent styling and an underline.

```js
#let title_section(title) = {
    set text(size: 14pt, weight: "semibold")
    align(left)[
    #smallcaps[#title]
    #line(length: 100%, stroke: (paint: gray, thickness: 1pt))
    ]
}
```

iii) **Component Functions:**

* Centers and styles the name of the individual.

```js
#let name_component(firstname, lastname) = {
    align(center)[
    #pad(bottom: 5pt)[
        #block[
        #set text(size: 25pt, style: "normal")
        #text(weight: "light")[#firstname]
        #text(weight: "light")[#lastname]
        ]
    ]
    ]
}
```

* This function generates a row of contact information with icons, demonstrating Typst’s ability to easily incorporate SVG icons.

```js
#let contacts_component(contact_info) = {
    set box(height: 11pt)
    set text(size: 11pt)

    let icons = (
    phone: box(image("../assets/icons/phone-solid.svg")),
    email: box(image("../assets/icons/envelope-solid.svg")),
    portfolio: box(image("../assets/icons/globe-solid.svg")),
    linkedin: box(image("../assets/icons/linkedin.svg")),
    github: box(image("../assets/icons/github.svg")),
    medium: box(image("../assets/icons/medium.svg"))
    )

    let links = (
    phone: "",
    email: "mailto:",
    portfolio: "https://www.",
    linkedin: "https://www.linkedin.com/in/",
    github: "https://github.com/",
    medium: "https://medium.com/@"
    )

    let separator = box(width: 5pt)
    
    align(center)[
    #block[
        #align(horizon)[
        #for (key, value) in contact_info {
            icons.at(key)
            separator
            box[#link(links.at(key) + value)[#value]]
            separator
        }
        ]
    ]
    ] 
}
```

* This function formats the work experience section, showcasing Typst’s ability to handle complex nested structures and iterating it using a loop.

```js
#let work_experience_component(experience_details) = {
    for (_, work) in experience_details {
    pad[
        #if (work.company != "" and work.location != "") [
        #justify_align[
            #set text(size: 11pt, style: "normal", weight: "semibold", fill: font_color_headings)
            #work.company
        ][
            #set text(size: 11pt, style: "normal", weight: "semibold", fill: font_color_headings)
            #work.location
        ]
        ]
        #justify_align[
        #set text(size: 9pt, style: "italic", weight: "semibold", fill: font_color_headings)
        #work.title
        ][
        #set text(size: 9pt, style: "normal", weight: "semibold", fill: font_color_headings)
        #work.duration
        ]
    ]

    set text(size: 10pt, style: "normal")
    set par(leading: 0.65em)
    work.work_summary
    }
}
```

iv) **Main Resume Function:**

* Defines the overall structure of the resume, including global settings and the inclusion of various sections.

```js
#let resume(author: (:), body) = {
    // Global settings
    set text(font: ("Roboto"), lang: "en", size: 10pt, fill: font_color, fallback: false)
    set page(paper: "a4", margin: (left: 0.30in, right: 0.30in, top: 0.20in, bottom: 0.20in))
    show par: set block(above: 0.75em, below: 0.75em)
    set par(justify: true)
    set heading(numbering: none, outlined: false)

    // Components
    name_component(author.firstname, author.lastname)
    positions_component(author.positions)
    contacts_component(author.contact)
    section_content("professional summary", author.professional_summary)
    section_content("work experience", work_experience_component(author.experience_details))
    section_content("certifications", author.certifications)
    section_content("skills", author.skills)
    section_content("education", author.education)
}
```

Generated resume in PDF format can be accessed [here](https://github.com/madhank93/typst-resume-template/blob/main/john_doe.pdf).

<div align="center">* * * *</div>

<center>

Originally published on [Medium](https://medium.com/@madhankumaravelu93/streamlined-resume-design-using-typst-b6cd603c97b3)

🌟 🌟 🌟 **The source code for this blog post can be found here** 🌟🌟🌟

[GitHub - madhank93/typst-resume-template](https://github.com/madhank93/typst-resume-template)

</center>

**Reference**:

* [https://typst.app/docs/](https://typst.app/docs/)

* [https://typst.app/docs/reference/syntax/](https://typst.app/docs/reference/syntax/)

* [https://typst.app/docs/reference/scripting/](https://typst.app/docs/reference/scripting/)