<!-- TITLE: Vue.js -->
<!-- SUBTITLE: Vue.js, JavaScript -->

# font awesome 임포트하기

```html
<style lang="scss">
@import ~@fontawesome/fontawesome-free/scss/fontawesome.scss 
</style>
```

이렇게하면 loader가 폰트파일을 못찾는다고 오류를 출력한다.
상대경로 처리 오류로 폰트 경로 sass변수를 수정해야 한다.
https://github.com/vuejs-templates/webpack/issues/1313#issuecomment-391393084
