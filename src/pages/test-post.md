---
title: This is a test post to make sure the styling is okay
href: /test-post
date: 2023-08-02
layout: ../layouts/PostLayout.astro
---
# Test post

[a link](https://www.google.com) and then some more text and some more text

### 1. Install

```
npm i whatever
```

### 2. Initialize

Initialize Firebase and add the `FirebaseApp` component to the component tree. Typically, this is done in the root `+layout.svelte` file to access Firebase in all pages.

#### +layout.svelte
```svelte
<script lang="ts">
    const app = initializeApp(/* your firebase config */);
    const db = getFirestore(app);
    const auth = getAuth(app);
</script>

<FirebaseApp {auth} {firestore}>
    <slot />
</FirebaseApp>
```