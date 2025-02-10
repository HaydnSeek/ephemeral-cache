# EphemeralCache - Auto-GC Memoization in JavaScript
EphemeralCache is a *lightweight, self-cleaning cache* for JavaScript that automatically removes unused values when they become unreachable. It leverages *WeakRef* and *FinalizationRegistry* to ensure memory efficiency.
## Features
* No manual cache managment - Entries expire automatically when not used
* Memory-efficient - Unlike `Map`, it won't retain unused objects
* Great for expensive computations - Ideal for caching API responses, heavy calculations, or large objects
## Usage
Add the following class to your project:
```
class EphemeralCache {
	constructor() {
		this.cache = new Map(); // Stores WeakRefs
		this.registry = new FinalizationRegistry((key) => {
			this.cache.delete(key); // Cleanup when object is GC'd
		});
	}

	memoize(key, computeFn) {
		let ref = this.cache.get(key);
		let value = ref?.deref(); // Try retrieving the value
		
		if (value === undefined) {
			value = computeFn(); // Compute if not in cache
			this.cache.set(key, new WeakRef(value)); // Store as WeakRef
			this.registry.register(value, key); // Attach cleanup
		}

		return value;
	}
}

export default EphemeralCache;
```
Then use it:
```
const cache = new EphemeralCache();

function heavyComputation(x) {
	console.log(`Computing for ${x}...`);
	return { result: x * 2 }; // Simulate expensive operation
}

// Memoizing
console.log(cache.memoize(5, () => heavyComputation(5))); // Computes
console.log(cache.memoize(5, () => heavyComputation(5))); // Retrieves from cache
```
