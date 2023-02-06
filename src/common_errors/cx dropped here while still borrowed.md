`argument requires that `cx` is borrowed for `'static`

This error occurs when cx (Scope) is used in a closure but that closure is missing the `move` keyword. As a result, the value is passed as a reference instead of copied into the closure. Adding  `move` before the closure that the `cx` is use in will usually fix this