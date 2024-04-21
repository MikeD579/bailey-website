<script setup lang="ts">
import { type Post } from '~/types/Post'
import { PortableText, type PortableTextComponents } from '@portabletext/vue'

const route = useRoute()

const query = groq`*[ _type == "post" && slug.current == $slug][0]`
const { data: post } = await useSanityQuery<Post>(query, {
  slug: route.params.slug,
})

const components: PortableTextComponents = {
  types: {
    image: ({ value }) => h('img', { src: value.imageUrl }),
  },

  marks: {
    link: ({ value }, { slots }) => {
      const target = (value?.href || '').startsWith('http') ? '_blank' : undefined;
      return h(
        'a',
        { href: value?.href, target, rel: target === '_blank' && 'external', style: 'text-decoration: none;' },
        slots.default?.(),
      );
    },
  },

  block: {
    // Ex. 1: customizing common block types
    h1: (_, { slots }) => h('h1', { class: 'text-2xl' }, slots.default?.()),
    blockquote: (_, { slots }) =>
      h('blockquote', { class: 'border-l-purple-500' }, slots.default?.()),

    // Ex. 2: rendering custom styles
    normal: (_, { slots }) =>
      h('p', { class: 'text-lg text-primary text-purple-700' }, slots.default?.()),
  },
};
</script>

<template>
  <section v-if="post" class="post">
    <img
      v-if="post.mainImage"
      class="post__cover"
      :src="$urlFor(post.mainImage).width(1920).url()"
      alt="Cover image"
    />
    <div v-else class="post__cover--none" />
    <div class="post__container">
      <h1 class="post__title">{{ post.title }}</h1>
      <p class="post__excerpt">{{ post.excerpt }}</p>
      <p class="post__date">{{ formatDate(post._createdAt) }}</p>
      <div v-if="post.body" class="portable__text">
        <PortableText :value="post.body" :components="components" />
      </div>
    </div>
  </section>
</template>

<style scoped>
.post {
  width: 100%;
  margin: var(--space-1) 0 var(--space-4);

  & .post__cover,
  & .post__cover--none {
    width: 100%;
    height: 200px;
    object-fit: cover;
    border-radius: 3px;
  }

  & .post__cover--none {
    background: var(--black);
  }

  & .post__container {
    padding: 0 var(--space-3);
  }

  & .post__title {
    font-family: var(--font-family-sans);
    font-size: var(--font-size-7);
    line-height: var(--line-height-6);
    margin: var(--space-4) 0;
    font-weight: 800;
  }

  & .post__excerpt {
    font-family: var(--font-family-serif);
    font-size: var(--font-size-5);
    line-height: var(--line-height-4);
    margin-top: 0;
    font-weight: 400;
  }

  & .post__date {
    font-family: var(--font-family-sans);
    font-weight: 600;
    font-family: var(--font-family-sans);
    font-size: var(--font-size-1);
    line-height: var(--line-height-1);
    margin-top: var(--space-4);
  }
}

@media (min-width: 800px) {
  .post {
    & .post__cover,
    & .post__cover--none {
      width: 750px;
      height: 380px;
    }

    & .post__title {
      font-size: var(--font-size-10);
      line-height: var(--line-height-10);
      margin: var(--space-6) 0 0;
      letter-spacing: -0.025em;
    }

    & .post__excerpt {
      font-size: var(--font-size-5);
      line-height: var(--line-height-5);
      margin-top: var(--space-3);
      margin-bottom: var(--space-3);
    }

    & .post__date {
      font-size: var(--font-size-3);
      line-height: var(--line-height-2);
      margin-top: var(--space-0);
    }

    & .post__content {
      margin-top: var(--space-7);
    }
  }
}
</style>
