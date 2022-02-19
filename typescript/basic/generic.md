# ☕ generics sample

---

#### for array.prototype.filter
```js
interface FilterFunction<T = any> {
	(val: T): boolean;
}

const filter: FilterFunction<string> = val => typeof val ===string
filter(0);//error
filter('abc');// ok

```