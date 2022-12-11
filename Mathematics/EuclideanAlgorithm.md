
유클리드 호제법은 두 수의 최대공약수를 구하는 데 특화된 알고리즘이다.

각 수를 소인수분해 하여 최대공약수를 구하는 것보다 계산하는 횟수가 적으므로 큰 수의 최대공약수를 구하는 데 효과적이다.

```swift
/// Calculate Greatest Common Divisor from two integers.
///
/// - Parameters:
///     - lh: Left hand parameter greater or equal than 0.
///     - rh: Right hand parameter MUST greater than 0.
func euclideanAlgorithm(
  _ lh: Int,
  _ rh: Int) -> Int {
  
  guard rh > 0 else { return 0 }
  
  var lh = lh
  var rh = rh
  var share = 0
  var remainder = 0
  
  while (rh > 0) {
    
    share = lh / rh
    remainder = lh - share * rh
    
    lh = rh
    rh = remainder
  }
  
  return lh
}
```
