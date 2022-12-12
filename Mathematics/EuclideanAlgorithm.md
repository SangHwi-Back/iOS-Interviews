
유클리드 호제법은 두 수의 최대공약수를 구하는 데 특화된 알고리즘이다.

각 수를 소인수분해 하여 최대공약수를 구하는 것보다 계산하는 횟수가 적으므로 큰 수의 최대공약수를 구하는 데 효과적이다.

유클리드 호제법은 다음의 정의를 기반으로 한다.

> 정의 1 : gcd(a, 0) = a
> 
> 정의 2 : gcd(a, b) = gcd(b, r). r은 a를 b로 나눈 나머지.

구현은 아래와 같다.

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
  
  var share: Int = 0
  var remainder: Int = 0
  
  while (rh > 0) {
    
    share = lh / rh
    remainder = lh - share * rh
    
    lh = rh
    rh = remainder
  }
  
  return lh
}
```

---

아래는 확장 유클리드 호제법이다. 다음의 식에 대해 s, t, gcd 를 구하는 방법이다.

> sa + tb = gcd(a, b)

갑자기 s와 t가 튀어나온 이유는 수의 역원을 구하는 방법과 선형 디오판투스 방정식을 구하는 데 유용하기 때문이다.

> 역원 : 어떤 수 a 와 e 를 연산한 결과가 a 가 될 때 e 를 항등원이라고 하고 a 와 a<sup>-1</sup> 을 연산한 결과가 e 가 될 때 a<sup>-1</sup> 을 연산에 대한 a 의 역원이라고 한다.
> 
> 선형 디오판투스 방정식 : a 와 b 는 최대공약수를 구하고 싶은 두 수이고 c 는 최대공약수일 때 다음의 식을 만족하는 x 와 y 의 해를 구하는 방정식이다. ax + by = c.

구현은 아래와 같다.

```swift
struct EuclideResult {
  let s: Int
  let t: Int
  let gcd: Int
  
  init(
    _ s: Int,
    _ t: Int,
    _ gcd: Int) {
    
    self.s = s
    self.t = t
    self.gcd = gcd
  }
  
  init() {
    self.init(0, 0, 0)
  }
}

func extendedEuclideanAlgorithm(
  _ lh: Int,
  _ rh: Int) -> EuclideResult {
  
  guard rh > 0 else { return EuclideResult() }
  
  var lh = lh
  var rh = rh
  
  var s = 0
  var s1 = 1
  var s2 = 0
  
  var t = 0
  var t1 = 0
  var t2 = 1
  
  var share: Int = 0
  var remainder: Int = 0
  
  while (rh > 0) {
    
    share = lh / rh
    
    remainder = lh - share * rh
    s = s1 - share * s2
    t = t1 - share * t2
    
    lh = rh
    rh = remainder
    
    s1 = s2
    s2 = s
    
    t1 = t2
    t2 = t
  }
  
  return EuclideResult(s1, t1, lh)
}
```

Reference:
* [도서] 스토리로 이해하는 암호화 알고리즘 - 김수민 지음