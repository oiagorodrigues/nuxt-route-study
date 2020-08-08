<template>
  <div>lesson route inside course/{{ this.$route.params.courseCode }}</div>
</template>

<script>
export default {
  validate({ params, query }) {
    return params.courseCode && query.lessonCode;
  },
};
</script>
