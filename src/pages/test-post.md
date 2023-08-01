---
title: This is a test post to make sure the styling is okay
href: /test-post
date: 1st Aug 2023
layout: ../layouts/Layout.astro
---
# QuickStart

SvelteFire works in both SvelteKit and standalone Svelte apps. This guide assumes you're using SvelteKit.


### 1. Install

```
npm i sveltefire firebase
```

### 2. Initialize

Initialize Firebase and add the `FirebaseApp` component to the component tree. Typically, this is done in the root `+layout.svelte` file to access Firebase in all pages.

#### +layout.svelte
```svelte
<script lang="ts">
    import { FirebaseApp } from 'sveltefire';
    import { initializeApp } from 'firebase/app';
    import { getFirestore } from 'firebase/firestore';
    import { getAuth } from 'firebase/auth';

    // Initialize Firebase
    const app = initializeApp(/* your firebase config */);
    const db = getFirestore(app);
    const auth = getAuth(app);
</script>

<FirebaseApp {auth} {firestore}>
    <slot />
</FirebaseApp>
```