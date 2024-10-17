---
layout: cover
backgroundOpacity: 0.2
# You can also start simply with 'default'
theme: seriph
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
background: ./assets/slide_1.jpg
# some information about your slides (markdown enabled)
title: Yewlish-fetch
info: |
  ## Yewlish-fetch avaya persentation
  Presentation slides for developers.

  Learn more at [Sli.dev](https://sli.dev)
# apply unocss classes to the current slide
class: text-left
# https://sli.dev/features/drawing
drawings:
  persist: false
# slide transition: https://sli.dev/guide/animations.html#slide-transitions
transition: slide-left
# enable MDC Syntax: https://sli.dev/features/mdc
mdc: true
# take snapshot for each slide in the overview
overviewSnapshots: true
---

# **What _yewlish-fetch_ is?**

## Its Importance and Necessity

<!-- Apply entrance animation to the definition -->
<!-- .element: class="animate__animated animate__fadeInUp animate__delay-1s" -->

**🧩 Formal definition:**

> yewlish-fetch is a Rust crate that provides a derive macro for generating Yew fetch API bindings. It simplifies the process of making HTTP requests in Yew applications by generating client code based on an enum definition.

Source: [official documentation](https://lib.rs/crates/yewlish-fetch)

<div class="abs-br m-6 flex gap-2">
  <a href="https://github.com/patchwork-body/yewlish/tree/yewlish-query/fetch" target="_blank" alt="GitHub" title="Open in GitHub"
    class="text-xl slidev-icon-btn opacity-50 !border-none !hover:text-white">
    <carbon-logo-github />
  </a>
</div>

---
transition: fade-out
---

# The old ways of making HTTP requests in Yew

````md magic-move {lines: true}
```rust {*|2|5-7}
// Define an async context
#[function_component(App)]
fn app() -> Html {
  use_effect(|| {
    spawn_local(async move {
      // Some code here...
    });
  });

  html! {
    <>
      // ...
    </>
  }
}
```

```rust {*|1-4|6-11|12-21}
// Http request with Fetch API
let mut opts = RequestInit::new();
opts.method("GET");
opts.mode(RequestMode::Cors);

let request = Request::new_with_str_and_init("https://api.example.com/data", &opts).unwrap();
request
  .headers()
  .set("Accept", "application/json")
  .unwrap();

// Fetch the data
let window = web_sys::window().unwrap();
let resp_value = JsFuture::from(window.fetch_with_request(&request)).await.unwrap();

// Convert to Response
let resp: Response = resp_value.dyn_into().unwrap();

// Parse JSON
let json = JsFuture::from(resp.json().unwrap()).await.unwrap();
// Handle the JSON data...
```

```rust
use gloo_net::http::Request;

let response = Request::get("https://api.example.com/data")
  .header("Accept", "application/json")
  .send()
  .await
  .unwrap();

// Parse JSON
if response.status() == 200 {
    let data: serde_json::Value = response.json().await.unwrap();
    // Handle the JSON data...
}
```

```rust {*|3|4|5|6}
#[function_component(App)]
fn app() -> Html {
  let data = use_state(|| None);
  let error = use_state(|| None);
  let loading = use_state(|| false);
  let abort_controller = use_state(|| AbortController::new());

  use_effect(|| {
    spawn_local(async move {
      // Some code here...
    });
  });

  html! {
    <>
      // ...
    </>
  }
}
```
````

<style>
h1 {
  background-color: #2B90B6;
  background-image: linear-gradient(45deg, #4EC5D4 10%, #146b8c 20%);
  background-size: 100%;
  -webkit-background-clip: text;
  -moz-background-clip: text;
  -webkit-text-fill-color: transparent;
  -moz-text-fill-color: transparent;
}
</style>

---
transition: fade-out
---

# Our first Fetch schema

````md magic-move {lines: true}
```rust {*|1-3|4-7}
// fetch_schema.rs
use yewlish_fetch::FetchSchema;

#[derive(FetchSchema)]
pub enum Api {
  #[get("/posts", query = PostFilter, res = Vec<Post>)]
  GetPosts,
}
```

```rust {8-10}
// fetch_schema.rs
use yewlish_fetch::FetchSchema;

#[derive(FetchSchema)]
pub enum Api {
  #[get("/posts", query = PostFilter, res = Vec<Post>)]
  GetPosts,
  #[get("/posts/{id}", slugs = PostSlug, res = Post)]
  GetPost,
}
```

```rust {10-13}
// fetch_schema.rs
use yewlish_fetch::FetchSchema;

#[derive(FetchSchema)]
pub enum Api {
  #[get("/posts", query = PostFilter, res = Vec<Post>)]
  GetPosts,
  #[get("/posts/{id}", slugs = PostSlug, res = Post)]
  GetPost,
  #[post("/posts", body = PostBody, res = Post)]
  CreatePost
}
```

```rust {12-15}
// fetch_schema.rs
use yewlish_fetch::FetchSchema;

#[derive(FetchSchema)]
pub enum Api {
  #[get("/posts", query = PostFilter, res = Vec<Post>)]
  GetPosts,
  #[get("/posts/{id}", slugs = PostSlug, res = Post)]
  GetPost,
  #[post("/posts", body = PostBody, res = Post)]
  CreatePost,
  #[put("/posts/{id}", slugs = PostSlug, body = PostBody, res = Post)]
  UpdatePost
}
```

```rust {14-17}
// fetch_schema.rs
use yewlish_fetch::FetchSchema;

#[derive(FetchSchema)]
pub enum Api {
  #[get("/posts", query = PostFilter, res = Vec<Post>)]
  GetPosts,
  #[get("/posts/{id}", slugs = PostSlug, res = Post)]
  GetPost,
  #[post("/posts", body = PostBody, res = Post)]
  CreatePost,
  #[put("/posts/{id}", slugs = PostSlug, body = PostBody, res = Post)]
  UpdatePost,
  #[delete("/posts/{id}", slugs = PostSlug)]
  DeletePost
}
```
````

<style>
h1 {
  background-color: #2B90B6;
  background-image: linear-gradient(45deg, #4EC5D4 10%, #146b8c 20%);
  background-size: 100%;
  -webkit-background-clip: text;
  -moz-background-clip: text;
  -webkit-text-fill-color: transparent;
  -moz-text-fill-color: transparent;
}
</style>

---
transition: fade-out
---

# Fetching data with Yewlish-fetch

````md magic-move {lines: true}
```rust
// main.rs
use yew::prelude::*;
use fetch_schema::*;

#[derive(Debug, Clone, PartialEq, Serialize)]
pub struct PostSlug {
  // Post fields...
}

#[derive(Debug, Clone, PartialEq, Serialize)]
pub struct PostFilter {
  // Post fields...
}

#[derive(Debug, Clone, PartialEq, Serialize)]
pub struct PostBody {
  // Post fields...
}

#[derive(Debug, Clone, PartialEq, Deserialize)]
pub struct Post {
  // Post fields...
}
```

```rust
#[function_component(App)]
fn app() -> Html {
  let client = ApiFetchClient::new("https://api.example.com");

  html! {
    <ApiFetchClientProvider client={client}>
      // Your components here...
    </ApiFetchClientProvider>
  }
}
```

```rust {4-12}
#[function_component(App)]
fn app() -> Html {
  let client = ApiFetchClient::new("https://api.example.com")
    .with_middlewares(vec![Rc::new(
      move |request_init, headers| {
        let future = async move {
          // Some code here...
        };

        Box::pin(future)
      }
    )]);

  html! {
    <ApiFetchClientProvider client={client}>
      // Your components here...
    </ApiFetchClientProvider>
  }
}
```

```rust {13-16}
#[function_component(App)]
fn app() -> Html {
  let client = ApiFetchClient::new("https://api.example.com")
    .with_middlewares(vec![Rc::new(
      move |request_init, headers| {
        let future = async move {
          // Some code here...
        };

        Box::pin(future)
      }
    )])
    .with_cache(Cache::new(CacheOptions {
      policy: Some(CachePolicy::StaleWhileRevalidate),
      max_age: Some(10.0 * 60.0 * 1000.0); // Ten minutes
    }));

  html! {
    <ApiFetchClientProvider client={client}>
      // Your components here...
    </ApiFetchClientProvider>
  }
}
```

```rust
pub enum CachePolicy {
    #[default]
    StaleWhileRevalidate, // Show cached data while fetching the new one
    CacheThenNetwork, // Show cached data if available and not expired otherwise fetch the new one
    NetworkOnly, // Always fetch the new data (ignore the cache)
    CacheOnly, // Always show the cached data (ignore the network)
}
```
````

<style>
h1 {
  background-color: #2B90B6;
  background-image: linear-gradient(45deg, #4EC5D4 10%, #146b8c 20%);
  background-size: 100%;
  -webkit-background-clip: text;
  -moz-background-clip: text;
  -webkit-text-fill-color: transparent;
  -moz-text-fill-color: transparent;
}
</style>

---
transition: fade-out
---

# Fetching data with Yewlish-fetch

````md magic-move {lines: true}
```rust
#[function_component(Test)]
pub fn test() -> Html {
  // get_posts.data
  // get_posts.error
  // get_posts.loading
  // get_posts.cancel
  let get_posts = use_get_posts(GetPostsParams::default());

  // All the same as above
  // + get_posts.trigger(GetPostsParams::default())
  let get_posts_async = use_get_posts_async();

  let get_posts_with_options = use_get_posts_with_options(
    GetPostsParams::default(), GetPostsOptions::default()
  );

  let get_posts_with_options_async = use_get_posts_with_options_async(GetPostsOptions::default());

  html! {
    <>
      // Your components here...
    </>
  }
}
```

```rust
#[function_component(Test)]
pub fn test() -> Html {
  let posts = use_get_posts_async();

  if let Some(error) = (*posts.error).clone() {
      return html! { format!("Error fetching posts: {error:?}") }
  }

  html! {
      <>
          <button onclick={Callback::from(move |_event: MouseEvent| {
              posts.trigger.emit(GetPostsParams::default());
          })}>{ "Fetch" }</button>

          {if *posts.loading {
              html! { "Loading..." }
          } else {
              html! {
                {for (*posts.data).clone().unwrap_or_default().iter().map(|post| html! {<></>})
              }
          }}
      </>
  }
}
```

```rust
#[function_component(Test)]
pub fn test() -> Html {
  let posts = use_get_posts(GetPostsParams {
    query: PostFilter { limit: 10 },
    ..Default::default()
  });

  html! {
      <>
        // Your components here...
      </>
  }
}
```

```rust {7-17}
#[function_component(Test)]
pub fn test() -> Html {
  let posts = use_get_posts_with_options(
    GetPostsParams {
      query: PostFilter { limit: 10 },
      ..Default::default()
    },

    GetPostsOptions {
      cache_options: Some(CacheOptions {
        policy: Some(CachePolicy::CacheOnly),
        ..Default::default()
      }),

      ..Default::default()
    });

  html! {
      <>
        // Your components here...
      </>
  }
}
```

```rust {7-15}
#[function_component(Test)]
pub fn test() -> Html {
  let posts = use_get_posts_with_options(
    GetPostsParams {
      query: PostFilter { limit: 10 },
      ..Default::default()
    },

    GetPostsOptions {
      on_success: Some(Callback::from(move |data: Vec<Post>| {
        // Handle the data...
      }),

      ..Default::default()
    });

  html! {
      <>
        // Your components here...
      </>
  }
}
```

```rust {7-15}
#[function_component(Test)]
pub fn test() -> Html {
  let posts = use_get_posts_with_options(
    GetPostsParams {
      query: PostFilter { limit: 10 },
      ..Default::default()
    },

    GetPostsOptions {
      on_error: Some(Callback::from(move |error: FetchError| {
        // Handle the error...
      }),

      ..Default::default()
    });

  html! {
      <>
        // Your components here...
      </>
  }
}
```
````

<style>
h1 {
  background-color: #2B90B6;
  background-image: linear-gradient(45deg, #4EC5D4 10%, #146b8c 20%);
  background-size: 100%;
  -webkit-background-clip: text;
  -moz-background-clip: text;
  -webkit-text-fill-color: transparent;
  -moz-text-fill-color: transparent;
}
</style>

---
transition: fade-out
---

# What's next?

- [ ] Different Cache policies
- [ ] Middlewares
- [ ] Smart refetching
- [ ] Advanced Cache manipulation
- [ ] Optimistic UI updates
- [ ] Garbage collection

<style>
h1 {
  background-color: #2B90B6;
  background-image: linear-gradient(45deg, #4EC5D4 10%, #146b8c 20%);
  background-size: 100%;
  -webkit-background-clip: text;
  -moz-background-clip: text;
  -webkit-text-fill-color: transparent;
  -moz-text-fill-color: transparent;
}
</style>

