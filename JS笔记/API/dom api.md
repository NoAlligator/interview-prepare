> ##### 将节点和选择描述字符串进行匹配：`Element.matches(selectorString: string): boolean`

```typescript
let node = document.querySelector('input#password')
let result = node?.matches('input#password')
//result is true
```

