<!-- TITLE: Webpack -->
<!-- SUBTITLE: Webpack, JavaScript -->

# Tree shaking
나무를 흔들어서 죽은 잎(코드)을 떨어뜨린다는 의미라고 한다. 이 장점을 온전히 누리려면,
* ES2015 module syntax를 사용해라. import/export
* package.json에 sideEffects 프로퍼티를 추가해라.
* webpack 설정파일에 mode 프로퍼티를 production으로 사용해라.
