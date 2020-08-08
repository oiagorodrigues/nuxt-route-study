<template>
  <div>unknown dynamic route inside course/{{ this.$route.params.courseCode }}</div>
</template>

<script>
export default {
  validate({ params }) {
    const [, lessonCode] = params.pathMatch.split('/');
    return params.courseCode && /^\d+$/g.test(lessonCode);
  },
};
</script>
