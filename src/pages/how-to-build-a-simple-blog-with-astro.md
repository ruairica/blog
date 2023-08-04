---
title: How to build a simple blog with Astro
href: /how-to-build-a-simple-blog-with-astro
date: 2023-08-04
layout: ../layouts/PostLayout.astro
---

I wanted to learn a new technology and built the site in a way where it would be easy to make new blog posts.
This post covers some of Astro's basic functionality that I used to build the main functionality of this site.
Having a basic understanding of HTML/CSS/Javsscript is probably required.

### Enter Astro

[Astro](https://astro.build/) is a javascript framework for making multi page applications mainly aimed at making content focused sites as it focuses heavily on a fast first page load and being all around SEO friendly.
Astro as a static site generator is quite simple, you write content in markdown, and then you can template the markdown into pages using ```.astro``` files.
Pages can have client-side interactivity with React/Svelte/Vue components embedded in Astro components which I will look at in the future but for now we're keeping it basic.


### What we're building

A site that supports writing posts in markdown, that are automatically linked to from the homepage in descending order of date.
I'll be using the default astro project as a base and making modifications to it.

### Getting Started

Before starting if you're using VS Code I'd recommend installing the "Astro" extension.

To get started run

```
npm create astro@latest
```

There's be a few options to for you to choose just make sure to select any option that is marked ```recommended```, for the other ones it's up to your preference.


```cd``` into the created folder and ```npm run dev``` will launch the site locally.

#### Astro Fundamentals

```index.astro``` is the home page of the site, you'll see it looks quite a lot like a normal html page, in fact it's a superset of html, but there's a few things to notice:


At the top of the page there is the frontmatter which is essentially the javascript for the page, here you can import other astro components and run some javascript which we'll look at shortly
```astro
---
import Layout from '../layouts/Layout.astro';
import Card from '../components/Card.astro';
---
```

There's two astro components being used on the homepage, ```Card``` and  ```Layout```

You can see that the card component is passed 3 props which can then be used within the component

```astro
<Card
    href="https://docs.astro.build/"
    title="Documentation"
    body="Learn how Astro works and explore the official API docs."
/>
```

Open  ```Card.astro```

Components can access their props with Astro.props in the front matter

```astro
const { href, title, body } = Astro.props;
```

and then interpolated into the html with curly brackets
```astro
<a href={href}>

```

Components also have their own style tags. Component styles are scoped to that component only by default, or can be made global with ```is:gloabl```

```astro
<style is:global></style>
```


The index.astro page content is wrapped with the Layout Component page```<Layout title="Welcome to Astro.">```. Layout Components are essentially a wrapper for your content,
you'll see in Layout.astro it contains the html head tag and body. index.astro (or any other component that uses the layout) will be placed where the

```astro
<slot />
```
is in the layout.

### Creating Content
Pages must be in the ```pages``` directory, they can be ```.astro``` or ```.md files```. Make a new file called ```my-first-blog.md``` in the Pages directory and write some text in it.

you can now visit the page at ```http://127.0.0.1:3000/my-first-blog```. You'll see Astro has transpiled the page to html for us.
Markdown pages can also have frontmatter. Insert this at the top of my-first-blog.md

```markdown
---
title: My First Blog!
href: /my-first-blog
date: 2023-08-02
layout: ../layouts/Layout.astro
---
```

title,href, and date are "normal" props, but we're using layout with a relative file path so that our my-first-blog page is also wrapped in the same layout as the home page.


We'll update a few things here to get some styling on our my-first-blog page ```index.astro``` cut

```css
main {
  margin: auto;
  padding: 1rem;
  width: 800px;
  max-width: calc(100% - 2rem);
  color: white;
  font-size: 20px;
  line-height: 1.6;
}
```

and paste it with the other css in ```Layout.astro```. Remove the opening and closing ```<main>``` tags from ```index.astro```, In ```Layout.astro``` wrap ```<slot />``` in main tags, also makes the styles global with ```is:global```, here's a snippet of our changes to Layout.astro

```astro
    <main>
      <slot />
    <main>
  </body>
</html>
<style is:global>
  main {
  margin: auto;
  padding: 1rem;
  width: 800px;
  max-width: calc(100% - 2rem);
  color: white;
  font-size: 20px;
  line-height: 1.6;
}
```




Let's re-purpose ```Card.astro``` slightly by renaming ```body``` to ```date``` as we'll use it to link to blog posts from the main page

```astro
---
interface Props {
  title: string;
  date: string;
  href: string;
}

const { href, title, date } = Astro.props;
---

<li class="link-card">
  <a href={href}>
    <h2>
      {title}
      <span>&rarr;</span>
    </h2>
    <p>
      {date}
    </p>
  </a>
</li>
```

Delete the `<Card>` tags from index.astro for now, and add this line to the frontmatter of index.astro

```javascript
const posts = await Astro.glob("./*.md");
```

This line retrieves all the .md files in the the ```pages``` directory, so everytime we add a new blog post it will automatically be retrieved here.

Inside the ```<ul>``` tag on ```index.astro``` (where we deleted the ```<Card>``` tags from) add

```astro
      {
        posts
          .sort(
            (a, b) =>
              new Date(b.frontmatter.date).valueOf() -
              new Date(a.frontmatter.date).valueOf()
          )
          .map((post) => (
            <Card
              href={post.frontmatter.href}
              title={post.frontmatter.title}
              date={Intl.DateTimeFormat().format(new Date(post.frontmatter.date))}
            />
          ))
      }
```

There's a few things going on here, first we open the curly brackets to let Astro know we're about to do some javascript, Astro has a JSX type syntax here you might be familiar with if you've used React. So we have the ```posts``` array from the front matter, we can access the props we set up earlier of the post by using ```.frontmatter.propName```.

So we sort the posts in descending date order and then map them to ```Card``` components, passing in the required props.
```javascript
Intl.DateTimeFormat().format(new Date(post.frontmatter.date))
```
is just to format from our YYYY-MM-DD format into the users local readable date format.
You can now click on the card to link directly to my-first-blog page, and all future blog pages will automatically be added as ```<Cards>``` on the home page!

I want to update the blog posts to display the dates and have a link back to the homepage.