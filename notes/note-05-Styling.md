# Styling
Next.js supports different ways of styling your application, including:
- Global CSS
  - for `pages` directory: can only import global styles inside the `_app.js` file
- CSS Modules
  - extension: `.module.css`
  - locally scope CSS by automatically creating a unique class name
- Tailwind CSS
  - [Install Tailwind CSS with Next.js](https://tailwindcss.com/docs/guides/nextjs)
- Sass
  - built-in support by Next.js
- CSS-in-JS
  - CSS-in-JS libraries which require runtime JavaScript are not currently supported in Server Components
  -  to style Server Components, we recommend using CSS Modules or other solutions that output CSS files, like PostCSS or Tailwind CSS