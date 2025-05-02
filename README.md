# 📊 Agentnoon Org Chart Visualizer

This project is a Vue + Tailwind application built as part of the Agentnoon coding test. It visualizes a large-scale organizational hierarchy using CSV data, and calculates insightful reporting metrics like descendant counts, IC cost, manager cost, and cost ratios.

---

## 🔧 Tech Stack

- **Vue 3** (Composition API, `<script setup>`)
- **Tailwind CSS** for layout and visual styling
- **PapaParse** for robust CSV parsing
- **No TypeScript**, per prompt instructions

---

## 🧠 Key Features

✅ Upload and parse the provided 40k+ row CSV  
✅ Expand/collapse org tree nodes interactively  
✅ Recursively count descendants per employee  
✅ Calculate:
- **IC Cost** – sum of all salaries of individual contributors  
- **Management Cost** – sum of all salaries of people with reports  
- **Total Cost** – total cost of a subtree (excluding self)  
- **Manager:IC Ratio** – clean 2-decimal display, edge-handled  

✅ Performance-optimized: only renders a limited tree depth to avoid crashing  
✅ Clean, simple UI (with emojis!) that reflects user-friendly visual hierarchy  

---

## 🧪 How to Use

1. Upload the provided CSV (`giga-corp.csv`)
2. Wait for the tree to render (CEO appears at the top)
3. Expand nodes to explore nested team structures
4. Hover/read values to understand cost breakdowns and ratios

---

## 💡 Design Notes

- Rendering is limited to 3 levels deep to preserve performance in-browser  
- Recursive computations are memoized via `computed()`  
- All logic is housed in a single recursive `OrgNode.vue` component  
- Salaries are parsed and sanitized (removing commas and formatting)  
- Tailwind used for responsive layout and intuitive hierarchy display

---

## 📦 Live Demo or Run Instructions

> You can run locally or upload this to:
- [Codesandbox](https://codesandbox.io)
- [Codepen](https://codepen.io)
- [Vercel](https://vercel.com)

Or simply open `index.html` in a Vite-ready environment.

---

## 🙋 About Me

Hamza Elmi – 3rd-year Computer Science student with a strong focus on clean architecture, performance, and practical design

---


## ✅ Summary

> ⚡ Built with clarity, recursion, and respect for UX  
> 📈 Tested with 40,000+ employee records  
> 💼 Focused on performance and clean presentation  

Thanks for the opportunity! :)


---
