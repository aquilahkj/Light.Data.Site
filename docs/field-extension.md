# Field Extension

When querying or aggregate data, it is often necessary to extract some part of data in the specified fields for querying or aggregate. For example, extract the date from the time field for statistics, This can be done using the method or property of the field. In addition number type field support using `+` (addition) `-` (subtraction) `*` (multiplication) `/` (division) `%` (mod) or mathematical operations. String types support the using `+` (concat) for string concatenation.

## DataTime Method

| Method | Introduce |
|:------|:------|
| ToString(string format) | Format datetime string, Such as `yyyy-MM-dd` |

| Property | Introduce |
|:------|:------|
| Date | Get Date |
| Year | Get year |
| Month | Get Month |
| Day | Get Day |
| Hour | Get Hour |
| Minute | Get Minute |
| Second | Get Second |
| Week | Get Week(Different database definitions may not be consistent) |
| DayOfWeek | Get Day Of Week |
| DayOfYear | Get Day Of Year |

## String Method

| Method | Introduce |
|:------|:------|
| Substring(int index, int length) | Substring string |
| IndexOf(int value, int index) | Get index of string |
| Replace(string oldString, string newString) | Replace string |
| ToLower() | Convert to lower |
| ToUpper() | Convert to upper |
| Trim() | Clear the space before and after |
| Concat(object obj1,object obj2,object obj3...) | Static method, concat string |

| Property | Introduce |
|:------|:------|
| Length | String length |

## Math Method

| 方法 | 说明 |
|:------|:------|
| Abs | the absolute value of a specified number. |
| Sign | a value that indicates the sign of a number. |
| Sin | the sine of the specified angle. |
| Cos | the cosine of the specified angle. |
| Tan | the tangent of the specified angle. |
| Atan | the angle whose tangent is the specified number. |
| ASin | the angle whose sine is the specified number. |
| ACos | the angle whose cosine is the specified number. |
| Atan2 | the angle whose tangent is the quotient of two specified numbers. |
| Ceiling | the smallest integral value greater than or equal to the specified number. |
| Floor |the largest integral value less than or equal to the specified number. |
| Round | Rounds a value to the nearest integer or to the specified number of fractional digits. |
| Truncate | Calculates the integral part of a number. |
| Log | the logarithm of a specified number. |
| Log10 | the base 10 logarithm of a specified number. |
| Exp | e raised to the specified power. |
| Pow | a specified number raised to the specified power. |
| Sqrt | the square root of a specified number. |
| Max | the larger of two numbers. |
| Min | the smaller of two numbers. |