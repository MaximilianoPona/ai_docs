# Tailwind CSS Documentation Website

This is the official documentation website for Tailwind CSS v4, built with Next.js 15 and leveraging the latest Tailwind CSS features. The site provides comprehensive documentation, framework installation guides, interactive examples, a blog, and community resources. It uses MDX for documentation content, React Server Components for rendering, and advanced features like syntax highlighting with Shiki, dynamic Open Graph images, and RSS feed generation.

The project employs a modern Next.js architecture with parallel routes, nested layouts, and TypeScript throughout. Documentation is organized hierarchically with sections covering core concepts, utility classes, and framework-specific integration guides. The site supports both light and dark modes, includes interactive resizable examples, and provides extensive API reference tables for all Tailwind utilities.

## Documentation API Functions

### getDocPageBySlug

Retrieves a documentation page by its slug, loading the MDX file dynamically and returning the component with metadata.

```typescript
import { getDocPageBySlug } from '@/app/(docs)/docs/api';

// Fetch a documentation page
const page = await getDocPageBySlug('installation');

if (page) {
  const { Component, title, description } = page;
  // Render: <Component />
  console.log(title); // "Installation"
  console.log(description); // "Setting up Tailwind CSS in your project."
}

// Returns null if page doesn't exist
const missing = await getDocPageBySlug('non-existent');
console.log(missing); // null
```

### getDocPageSlugs

Returns an array of all available documentation page slugs by scanning the docs directory.

```typescript
import { getDocPageSlugs } from '@/app/(docs)/docs/api';

// Get all documentation page slugs
const slugs = await getDocPageSlugs();
console.log(slugs);
// ['installation', 'editor-setup', 'compatibility', 'aspect-ratio', 'columns', ...]

// Use for generating static paths
export async function generateStaticParams() {
  const slugs = await getDocPageSlugs();
  return slugs.map((slug) => ({ slug }));
}
```

### generateTableOfContents

Generates a table of contents from a documentation page by parsing heading structures from the MDX file.

```typescript
import { generateTableOfContents } from '@/app/(docs)/docs/api';
import type { TOCEntry } from '@/components/table-of-contents';

// Generate TOC for a documentation page
const toc = await generateTableOfContents('hover-focus-and-other-states');

console.log(toc);
// [
//   { level: 2, text: 'Pseudo-classes', slug: '#pseudo-classes', children: [...] },
//   { level: 2, text: 'Pseudo-elements', slug: '#pseudo-elements', children: [...] },
//   { level: 0, text: 'Quick reference', slug: '#quick-reference', children: [] }
// ]

// Render TOC
toc.forEach(entry => {
  console.log(`${entry.level === 0 ? '' : '  '.repeat(entry.level - 1)}${entry.text}`);
});
```

### getSectionAndTitleBySlug

Finds the section and title for a documentation page by searching through the navigation index.

```typescript
import { getSectionAndTitleBySlug } from '@/app/(docs)/docs/api';

// Get section and title for a page
const info = getSectionAndTitleBySlug('aspect-ratio');

if (info) {
  console.log(info.section); // "Layout"
  console.log(info.title);   // "aspect-ratio"
}

// Returns null if not found
const missing = getSectionAndTitleBySlug('invalid-slug');
console.log(missing); // null
```

## Blog API Functions

### getBlogPostBySlug

Retrieves a blog post by its slug, loading the MDX file and returning the component with full metadata.

```typescript
import { getBlogPostBySlug } from '@/app/blog/api';

// Fetch a blog post
const post = await getBlogPostBySlug('tailwindcss-v4');

if (post) {
  const { Component, meta, slug } = post;
  console.log(meta.title);       // "Tailwind CSS v4.0"
  console.log(meta.date);        // "2024-03-15"
  console.log(meta.description); // "An exciting new version..."
  console.log(meta.authors);     // [{ name: "Adam Wathan", twitter: "adamwathan" }]
  console.log(meta.image?.src);  // "/img/blog/post.jpg"
  console.log(meta.private);     // false
  // Render: <Component />
}

// Returns null for missing or inaccessible posts
const missing = await getBlogPostBySlug('non-existent');
console.log(missing); // null
```

### getBlogPostSlugs

Returns an array of all blog post slugs sorted by date (newest first), excluding private posts.

```typescript
import { getBlogPostSlugs } from '@/app/blog/api';

// Get all blog post slugs in reverse chronological order
const slugs = await getBlogPostSlugs();
console.log(slugs);
// ['tailwindcss-v4', 'headless-ui-v2', 'tailwind-ui-update', ...]

// Use for pagination or listing
export async function BlogIndex() {
  const slugs = await getBlogPostSlugs();
  const posts = await Promise.all(
    slugs.slice(0, 10).map(getBlogPostBySlug)
  );
  return posts.filter(Boolean);
}
```

### formatDate

Formats a timestamp string into a human-readable date format (e.g., "March 15, 2024").

```typescript
import { formatDate } from '@/app/blog/api';

// Format various date formats
const formatted1 = formatDate('2024-03-15');
console.log(formatted1); // "March 15, 2024"

const formatted2 = formatDate('2024-12-01T10:30:00Z');
console.log(formatted2); // "December 1, 2024"

// Use in blog post display
const post = await getBlogPostBySlug('some-post');
if (post) {
  const displayDate = formatDate(post.meta.date);
}
```

## Component API

### CodeExample

Renders syntax-highlighted code with optional filename and copy functionality using Shiki.

```typescript
import { CodeExample, js, html, css } from '@/components/code-example';

// Basic JavaScript example
<CodeExample
  example={js`
    const config = {
      theme: {
        extend: {
          colors: { primary: '#3b82f6' }
        }
      }
    };
  `}
  copyable={true}
/>

// With filename
<CodeExample
  example={html`
    <div class="bg-blue-500 text-white p-4">
      Hello World
    </div>
  `}
  filename="index.html"
  copyable={true}
/>

// CSS example
<CodeExample
  example={css`
    @import "tailwindcss";

    @theme {
      --color-primary: #3b82f6;
    }
  `}
  filename="styles.css"
  className="mt-4"
/>
```

### CodeExampleGroup

Groups multiple code examples with tabs for different file types or variations.

```typescript
import { CodeExampleGroup, CodeBlock, jsx, css } from '@/components/code-example';
import { TabPanel } from '@headlessui/react';

<CodeExampleGroup
  filenames={['App.jsx', 'styles.css', 'tailwind.config.js']}
>
  <CodeBlock example={jsx`
    export default function App() {
      return <div className="bg-primary text-white">Hello</div>
    }
  `} />
  <CodeBlock example={css`
    @import "tailwindcss";
  `} />
  <CodeBlock example={js`
    export default {
      theme: { extend: { colors: { primary: '#3b82f6' } } }
    }
  `} />
</CodeExampleGroup>
```

### ApiTable

Displays a utility class reference table with expandable rows for comprehensive API documentation.

```typescript
import { ApiTable } from '@/components/api-table';

// Render API table with utility classes
<ApiTable rows={[
  ['aspect-auto', 'aspect-ratio: auto;'],
  ['aspect-square', 'aspect-ratio: 1 / 1;'],
  ['aspect-video', 'aspect-ratio: 16 / 9;'],
  ['aspect-[4/3]', 'aspect-ratio: 4 / 3;'],
  // ... more rows
]} />

// Tables with > 15 rows automatically get "Show more" button
<ApiTable rows={[
  ['text-xs', 'font-size: 0.75rem; /* 12px */ line-height: 1rem; /* 16px */'],
  ['text-sm', 'font-size: 0.875rem; /* 14px */ line-height: 1.25rem; /* 20px */'],
  ['text-base', 'font-size: 1rem; /* 16px */ line-height: 1.5rem; /* 24px */'],
  // ... 20+ more entries will show first 10 with expand button
]} />

// Multiple CSS declarations per utility
<ApiTable rows={[
  ['bg-gradient-to-r', [
    'background-image: linear-gradient(to right, var(--tw-gradient-stops));',
    '--tw-gradient-from: transparent;',
    '--tw-gradient-to: transparent;'
  ]]
]} />
```

### Example

Renders an interactive example container with optional resizable functionality for responsive testing.

```typescript
import { Example } from '@/components/example';

// Basic example with padding
<Example>
  <div className="bg-blue-500 text-white p-4 rounded-lg">
    Sample component
  </div>
</Example>

// Resizable example for responsive design testing
<Example resizable={true}>
  <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
    <div className="bg-blue-500 p-4">Item 1</div>
    <div className="bg-green-500 p-4">Item 2</div>
    <div className="bg-red-500 p-4">Item 3</div>
  </div>
</Example>

// Without padding, custom styling
<Example padding={false} className="border-2 border-gray-300">
  <img src="/demo.jpg" alt="Demo" />
</Example>
```

### NewsletterForm

Renders an email subscription form for the Tailwind CSS newsletter with styling and validation.

```typescript
import { NewsletterForm } from '@/components/newsletter-form';

// Basic newsletter form
<NewsletterForm action="https://your-mailchimp-url.com/subscribe" />

// Usage in a page
export default function NewsletterSection() {
  return (
    <section className="bg-gray-50 dark:bg-gray-900 p-8">
      <h2 className="text-2xl font-bold mb-4">Stay Updated</h2>
      <p className="mb-6">Get the latest Tailwind CSS news.</p>
      <NewsletterForm action="https://api.example.com/subscribe" />
    </section>
  );
}
```

## API Routes

### Open Graph Image Generation

Dynamic Open Graph image generation route that creates preview images for any page.

```typescript
// Route: /api/og
// Usage: /api/og?path=/docs/installation

// Generates OG image for documentation page
fetch('https://tailwindcss.com/api/og?path=/docs/hover-focus-and-other-states')
  .then(res => res.blob())
  .then(blob => {
    // Returns 1200x630 PNG image with page title and description
  });

// Use in meta tags
export async function generateMetadata({ params }) {
  return {
    openGraph: {
      images: [`/api/og?path=/docs/${params.slug}`]
    }
  };
}

// Supports blog posts, documentation, and any site page
// Automatically extracts title, description, and section from page HTML
// Returns 404 error for non-existent pages
```

### RSS/Atom Feed Generation

Blog feed generation supporting multiple formats (RSS, JSON, Atom).

```typescript
// Routes:
// GET /feeds/feed.xml  - RSS 2.0 feed
// GET /feeds/feed.json - JSON Feed
// GET /feeds/atom.xml  - Atom 1.0 feed

// Subscribe to RSS feed
const response = await fetch('https://tailwindcss.com/feeds/feed.xml');
const xml = await response.text();
// Parse RSS XML for blog posts

// JSON Feed format (easier to parse)
const jsonResponse = await fetch('https://tailwindcss.com/feeds/feed.json');
const feed = await jsonResponse.json();
console.log(feed.items);
// [
//   {
//     id: "Tailwind CSS v4.0",
//     title: "Tailwind CSS v4.0",
//     url: "https://tailwindcss.com/blog/tailwindcss-v4",
//     date_published: "2024-03-15T00:00:00.000Z",
//     authors: [{ name: "Adam Wathan", url: "https://twitter.com/@adamwathan" }],
//     image: "https://tailwindcss.com/img/blog/tailwindcss-v4.jpg"
//   },
//   ...
// ]

// Atom feed
const atomResponse = await fetch('https://tailwindcss.com/feeds/atom.xml');
const atom = await atomResponse.text();
// Parse Atom XML
```

## Configuration

### Next.js Configuration

Next.js configuration with Tailwind CSS v4, MDX support, and custom webpack rules.

```typescript
// next.config.ts
import type { NextConfig } from "next";

const nextConfig: NextConfig = {
  // External packages for server-side rendering
  serverExternalPackages: ["@tailwindcss/node"],

  // Support MDX files as pages
  pageExtensions: ["js", "jsx", "ts", "tsx", "mdx"],

  // Experimental features
  experimental: {
    mdxRs: true, // Fast Rust-based MDX compiler
    turbo: {
      rules: {
        // Import SVG as React components
        "*.react.svg": {
          loaders: ["@svgr/webpack"],
          as: "*.js"
        }
      }
    }
  },

  // Redirects for deprecated/moved pages
  async redirects() {
    return [
      { source: "/docs", destination: "/docs/installation/using-vite", permanent: false },
      { source: "/docs/utility-first", destination: "/docs/styling-with-utility-classes", permanent: false }
    ];
  },

  // Proxy Tailwind UI content
  async rewrites() {
    return ["plus", "plus-assets", "vendor", "nova-api"].flatMap((path) => [
      { source: `/${path}`, destination: `https://tailwindui.com/${path}` },
      { source: `/${path}/:path*`, destination: `https://tailwindui.com/${path}/:path*` }
    ]);
  }
};

const withMDX = require("@next/mdx")();
module.exports = withMDX(nextConfig);
```

### Tailwind CSS Configuration

Tailwind CSS setup using the new v4 PostCSS plugin and CSS imports.

```typescript
// package.json
{
  "postcss": {
    "plugins": {
      "@tailwindcss/postcss": {}
    }
  }
}

// src/app/globals.css
@import "tailwindcss";

@theme {
  /* Custom theme variables */
  --color-primary: #3b82f6;
  --font-sans: 'Inter', sans-serif;
}

/* Custom utilities */
@utility tab-* {
  tab-size: *;
}

// Usage in components
import './globals.css';

export default function RootLayout({ children }) {
  return (
    <html lang="en">
      <body className="bg-white dark:bg-gray-950 text-gray-950 dark:text-white">
        {children}
      </body>
    </html>
  );
}
```

## Utility Functions

### stripShikiComments

Removes Shiki syntax highlighting control comments from code before copying to clipboard.

```typescript
import { stripShikiComments } from '@/components/shiki';

// Code with Shiki annotations
const code = `
function example() {
  // [!code highlight]
  const important = true;
  // [!code --]
  const removed = false;
  return important;
}
`;

const cleaned = stripShikiComments(code);
console.log(cleaned);
// Output:
// function example() {
//   const important = true;
//   return important;
// }

// Supports various comment styles
const htmlCode = `<!-- [!code highlight] --><div>Hello</div>`;
const cssCode = `/* [!code --] */color: red;`;
const shellCode = `# [!code highlight]\nnpm install`;

// Used automatically by CopyButton component
<CopyButton value={stripShikiComments(example.code)} />
```

## Summary

This Tailwind CSS documentation website demonstrates modern Next.js development patterns and serves as both comprehensive documentation and a showcase of Tailwind CSS v4 capabilities. The project architecture emphasizes type safety with TypeScript, server-side rendering for optimal performance, and component reusability. Key features include MDX-based content management, dynamic route generation, syntax highlighting, and automated SEO optimization through Open Graph images.

The site integrates multiple data sources and presentation layers: documentation pages with hierarchical navigation and table of contents generation, blog posts with RSS feed syndication, framework-specific installation guides with interactive code examples, and responsive component demonstrations. The build system leverages Next.js 15's latest features including React Server Components, parallel routes, and MDX compilation, while the Tailwind CSS v4 integration showcases the new PostCSS plugin architecture and CSS-based configuration system.
