---
title: 刨刨 Carbon API
date: 2017-05-23 17:25:38
description: 介绍 PHP 二次封装的时间库 Carbon，并列出 API；
tags:
- php-plugin
categories:
- PHP
---

# 介绍
`Carbon`是对`PHP DateTime`模块的二次扩展；提供时间格式化，时间计算的功能；

- 官方主页为 [http://carbon.nesbot.com/](http://carbon.nesbot.com/); 
- `Github`地址为 [https://github.com/briannesbitt/Carbon](https://github.com/briannesbitt/Carbon); 

# 文件结构
| 目录 | 描述 |
|: --- :|: --- :|
|-- src | `Carbon`源文件 |
|-- src\\Carbon | `Carbon`源文件 |
|-- src\\Carbon\\CarbonInterval.php | `DateInterval`类的二次扩展类`CarbonInterval`；主要用于时差计算；  |
|-- src\\Carbon\\Carbon.php | `DateTime`类的二次扩展类`Carbon`；提供时间计算，格式化输出的功能； |
|-- src\\Carbon\\Exceptions | 自定义异常文件夹  |
|-- src\\Carbon\\Lang | 语言本地化文件夹；`Carbon`类的`diffForHumans`方法会用到； |
| | |
|-- tests | `Carbon`测试用例文件 |
|-- tests\\AbstractTestCase.php | 所有测试文件的父类；提供了`执行前初始化`和`执行后清理`的功能, 及其它公共的API； |
|-- tests\\Carbon | 针对`src\\Carbon\\Carbon.php`的测试用例组 |
|-- tests\\CarbonInterval | 针对`src\\Carbon\\CarbonInterval.php`的测试用例组 |
|-- tests\\Localization | 针对`src\\Carbon\\Lang`的测试用例组 |

# API 细则
本篇涉及 `API` 为 `Carbon` 1.22.1 版本；
## Carbon
### 用途：生成`Carbon`实例
| 方法名 | 参数 | 描述 |
|: --- :|: --- :|: --- :|
| __construct | $time(null), $tz(null) | 根据格式化时间字符串和指定时区, 创建`Carbon`实例 |
| instance（static）| DateTime $dt| 根据 `DateTime`实例创建`Carbon`实例 |
| parse（static）| $time(null), $tz(null) | 根据格式化时间字符串和指定时区, 创建`Carbon`实例|
| create（static） | $year(null), $month(null), $day(null), $hour(null), $minute(null), $second(null), $tz(null) | 根据日期和时间创建`Carbon`实例 如果指定参数为`null`，会默认使用当前时间的对应值 |
| createSafe（static） | $year(null), $month(null), $day(null), $hour(null), $minute(null), $second(null), $tz(null) | 根据日期和时间创建`Carbon`实例 如果指定参数为`null`，会默认使用当前时间的对应值; 指定参数不符合规范，会返回异常； |
| createFromDate（static） | $year(null), $month(null), $day(null), $tz(null)| 根据日期创建`Carbon`实例如果指定参数为`null`，会默认使用当前时间的对应值|
| createFromTime（static） | $hour(null), $minute(null), $minute(null), $tz(null)| 根据时间创建`Carbon`实例如果指定参数为`null`，会默认使用当前时间的对应值|
| createFromFormat（static）| $format, $time, $tz（null） | 根据时间字符串及其对应的`format`字符串创建`Carbon`实例 |
| createFromTimestamp（static） | $timestamp, $tz(null) |根据时间戳和指定时区, 创建`Carbon`实例 |
| createFromTimestampUTC（static）| $timestamp| 根据时间戳和`utc`时区, 创建`Carbon`实例|
| now（static） | $tx(null) | 根据当前时间创建`Carbon`实例 |
| today（static） | $tx(null)| 根据当前时间创建`Carbon`实例，时间重置为 0时0分0秒 |
| tomorrow（static） | $tx(null)| 根据当前时间，加一天，创建`Carbon`实例|
| yesterday（static） | $tx(null)| 根据当前时间， 减一天， 创建`Carbon`实例 |
| minValue（static） |  | 创建系统支持的最小时间，并返回`Carbon`实例 |
| maxValue（static） | | 创建系统支持的最大时间，并返回`Carbon`实例 |
| copy | | 复制当前`Carbon`实例 |
| fromSerialized（static）| $value | 解析序列化字符串，创建`Carbon`实例 |

### 用途：修改`Carbon`实例
| 方法名 | 参数 | 描述 |
|: --- :|: --- :|: --- :|
| setDate | $year, $month, $day | 设置当前实例的年，月，日 |
| setDateTime | $year, $month, $day, $hour, $minute, $second(0) | 设置当前实例的年，月，日，时，分，秒|
| setTimeFromTimeString | $time| 根据 `H:i:s` 字符串设置当前实例时间 |
| timestamp | $value | 根据时间戳设置当前实例时间 | 
| second | $value | 设置当前实例时间指定秒 |
| minute | $value | 设置当前实例时间指定分钟|
| hour | $value | 设置当前实例时间指定小时|
| day | $value | 设置当前实例时间指定天|
| month | $value | 设置当前实例时间指月份 |
| year | $value |设置当前实例时间指定年份 |
| startOfDay | | 重置当前实例时间为 0时0分0秒 |
| endOfDay | | 重置当前实例时间为 23时59分59秒 |
| startOfWeek | | 重置当前实例时间为 本周的第一天，同时设置 0时0分0秒 |
| endOfWeek | | 重置当前实例时间为 本周的最后一天，同时设置 23时59分59秒 |
| startOfMonth | | 重置当前实例时间为 本月第一天，同时设置 0时0分0秒 |
| endOfMonth | | 重置当前实例时间为 本月最后一天，同时设置 23时59分59秒 |
| startOfQuarter | | 重置当前实例时间为 本季度第一天，同时设置 0时0分0秒 |
| endOfQuarter | | 重置当前实例时间为 本季度最后一天，同时设置 23时59分59秒 |
| startOfYear | | 重置当前实例时间为 本年第一天，同时设置 0时0分0秒 |
| endOfYear | | 重置当前实例时间为 本年最后一天，同时设置 23时59分59秒 |
| startOfDecade | | 重置当前实例时间为 所在十年的第一天，同时设置 0时0分0秒 |
| endOfDecade | | 重置当前实例时间为 所在十年的最后一天，同时设置 23时59分59秒 |
| startOfCentury | | 重置当前实例时间为 本世纪的第一天，同时设置 0时0分0秒 |
| endOfCentury | | 重置当前实例时间为 本世纪的最后一天，同时设置 23时59分59秒 |
| next  | $dayOfWeek(null)| 重置当前实例时间为 下一个指定`dayOfWeek`，同时设置 0时0分0秒 |
| previous  | $dayOfWeek(null) | 重置当前实例时间为 上一个指定`dayOfWeek`，同时设置 0时0分0秒 |
| nextWeekday  | | 重置当前实例时间为 下一个工作日，同时设置 0时0分0秒 |
| previousWeekday  | | 重置当前实例时间为 上一个工作日，同时设置 0时0分0秒 |
| nextWeekendDay  | | 重置当前实例时间为 下一个双休日，同时设置 0时0分0秒 |
| previousWeekendDay  | | 重置当前实例时间为 上一个双休日，同时设置 0时0分0秒 |
| firstOfMonth | $dayOfWeek(null) | 重置当前实例时间为 本月第一周的指定`dayOfWeek`，同时设置 0时0分0秒 |
| nthOfMonth   | $nth, $dayOfWeek | 重置当前实例时间为 本月第`n`周的指定`dayOfWeek`，同时设置 0时0分0秒 |
| lastOfMonth  | $dayOfWeek(null) | 重置当前实例时间为 本月最后一周的指定`dayOfWeek`，同时设置 0时0分0秒 |
| firstOfQuarter  | $dayOfWeek(null)| 重置当前实例时间为 当前季度第一周的指定`dayOfWeek`，同时设置 0时0分0秒 |
| nthOfQuarter  | $nth, $dayOfWeek | 重置当前实例时间为 当前季度第`n`周的指定`dayOfWeek`，同时设置 0时0分0秒 |
| lastOfQuarter  | $dayOfWeek(null)| 重置当前实例时间为 当前季度最后一周的指定`dayOfWeek`，同时设置 0时0分0秒 |
| firstOfYear  | $dayOfWeek(null)| 重置当前实例时间为 本年第一周的指定`dayOfWeek`，同时设置 0时0分0秒 |
| nthOfYear  | $nth, $dayOfWeek| 重置当前实例时间为 本年第`n`周的指定`dayOfWeek`，同时设置 0时0分0秒 |
| lastOfYear  | $dayOfWeek(null)| 重置当前实例时间为 本年最后一周的指定`dayOfWeek`，同时设置 0时0分0秒 |
| average | Carbon $dt | 重置当前实例时间为 与指定`Carbon`对象的中间时刻 |
| modify | $modify | 按`modify`字符串重置当前实例时间 |


### 用途：格式化时间
| 方法名 | 参数 | 描述 |
|: --- :|: --- :|: --- :|
| __toString | | 按变量`$toStringFormat`的格式输出 |
| toDateString | | 日期格式化输出 |
| toTimeString | | 时间格式化输出 |
| toDateTimeString | | 日期时间格式化输出 |
| toDayDateTimeString | | 格式化输出如 "Fri, Jan 3, 2013 10:50 PM" |
| toAtomString | | 格式化输出如 "2012-10-20T14:12:26+00:00"|
| toCookieString | | 格式化输出如 "Friday, 02-Jan-2012 14:20:39 UTC"|
| toIso8601String | | 同 `toAtomString`|
| toRfc822String | | 格式化输出如 "Mon, 15 Aug 05 15:52:01 +0000"|
| toRfc850String | | 格式化输出如 "Monday, 15-Aug-05 15:52:01 UTC"|
| toRfc1036String | | 格式化输出如 "2005-08-15T15:52:01+0000"|
| toRfc1123String | | 格式化输出如 "Mon, 15 Aug 2005 15:52:01 +0000"|
| toRfc2822String | | 格式化输出如 "Mon, 15 Aug 05 15:52:01 +0000"|
| toRfc3339String | | 同 `toAtomString` |
| toRssString | | 格式化输出如 "Mon, 15 Aug 2005 15:52:01 +0000"|
| toW3cString | | 格式化输出如 "2005-08-15T15:52:01+00:00"|
| toFormattedDateString | | 格式化输出如 "Jan 11, 1999" |
| formatLocalized| $format | 指定格式本地化输出 |

### 用途：时间判断
| 方法名 | 参数 | 描述 |
|: --- :|: --- :|: --- :|
| eq | Carbon $dt | 判断当前`Carbon`实例与指定`Carbon`对象时间是否一样 |
| equalTo | Carbon $dt  | 同 `eq` 方法|
| ne | Carbon $dt | 判断当前`Carbon`实例与指定`Carbon`对象时间是否不相同 |
| notEqualTo | Carbon $dt | 同 `ne` 方法 |
| gt | Carbon $dt | 判断当前`Carbon`实例是否大于指定`Carbon`对象时间 |
| greaterThan | Carbon $dt | 同 `gt` 方法 |
| gte | Carbon $dt | 判断当前`Carbon`实例是否大于等于指定`Carbon`对象时间 |
| greaterThanOrEqualTo | Carbon $dt | 同 `gte` 方法 |
| lt | Carbon $dt | 判断当前`Carbon`实例是否小于指定`Carbon`对象时间 |
| lessThan | Carbon $dt | 同 `lt` 方法 |
| lte | Carbon $dt | 判断当前`Carbon`实例是否小于等于指定`Carbon`对象时间 |
| lessThanOrEqualTo | Carbon $dt | 同 `lte` 方法 |
| between | Carbon $dt1, Carbon $dt2, $equal(true) | 判断当前`Carbon`实例是否在指定`Carbon`对象时间之间， 第三个参数表示是否可以等于指定`Carbon`对象 |
| min | Carbon $dt | 获取当前实例与指定`Carbon`对象中，最小的对象 |
| minimum| Carbon $dt | 同`min`|
| max| Carbon $dt | 获取当前实例与指定`Carbon`对象中，最大的对象 |
| maximum| Carbon $dt | 同`max` |
| closest | Carbon $dt1, Carbon $dt2 | 获取最接近当前实例时间的`Carbon`对象 |
| farthest | Carbon $dt1, Carbon $dt2 | 获取最不接近当前实例时间的`Carbon`对象 |
| isSameAs | $format, Carbon $dt | 判断当前实例与指定`Carbon`对象的`format`格式化结果是否相同|
| isSameDay | Carbon $dt | 判断当前实例与指定`Carbon`对象是否是同一天 |
| isSameMonth | Carbon $dt(null), $ofSameYear(false) | 判断当前实例与指定`Carbon`对象月份是否相同 |
| isBirthday | Carbon $dt | 判断当前实例与指定`Carbon`对象月日数是否相同 |
| isSameYear | Carbon $dt | 判断当前实例与指定`Carbon`对象年份是否相同 |
| isSunday | | 判断当前实例是否是周日 |
| isMonday | | 判断当前实例是否是周一 |
| isTuesday | | 判断当前实例是否是周二 |
| isWednesday | | 判断当前实例是否是周三 |
| isThursday | | 判断当前实例是否是周四 |
| isFriday | | 判断当前实例是否是周五 |
| isSaturday | | 判断当前实例是否是周六 |
| isYesterday | | 判断当前实例是否是昨天 |
| isToday | | 判断当前实例是否是今天 |
| isTomorrow | | 判断当前实例是否是明天 |
| isWeekday | | 判断当前实例是否属于工作日 |
| isWeekend | | 判断当前实例是否属于周末双休 |
| isLastWeek | | 判断当前实例是否属于上周 |
| isNextWeek | | 判断当前实例是否属于下一周 |
| isLastMonth | | 判断当前实例是否属于上一个月 |
| isCurrentMonth | | 判断当前实例是否属于当前月 |
| isNextMonth | | 判断当前实例是否属于下一个月 |
| isLastYear | | 判断当前实例是否属于去年 |
| isCurrentYear | | 判断当前实例是否属于当前年 |
| isNextYear | | 判断当前实例是否属于下一年 |
| isLeapYear | | 判断当前实例是否属于闰年 |
| isLongYear | | 判断当前实例是否属于长年，即一年不只有52个星期 |
| isPast | | 判断当前实例是否属于过去 |
| isFuture | | 判断当前实例是否属于未来 |



### 用途：时间计算
| 方法名 | 参数 | 描述 |
|: --- :|: --- :|: --- :|
| addSecond | $value(1) | 当前实例添加指定数量的秒数，返回当前实例 |
| subSecond | $value(1)| 当前实例减去指定数量的秒数，返回当前实例 |
| addSeconds | $value| 同`addSecond` |
| subSeconds | $value | 同`subSecond`|
| addMinute | $value(1)| 当前实例添加指定数量的分钟数，返回当前实例|
| subMinute | $value(1)| 当前实例减去指定数量的分钟数，返回当前实例|
| addMinutes | $value| 同`addMinute` |
| subMinutes | $value| 同`subMinute`|
| addHour | $value(1)| 当前实例添加指定数量的小时数，返回当前实例|
| subHour | $value(1)| 当前实例减去指定数量的小时数，返回当前实例|
| addHours | $value| 同`addHour`|
| subHours | $value| 同`subHour`|
| addDay | $value(1)| 当前实例添加指定数量的天数，返回当前实例|
| subDay | $value(1)| 当前实例减去指定数量的天数，返回当前实例|
| addDays | $value| 同`addDay`|
| subDays | $value| 同`subDay`|
| addWeekday | $value(1)| 当前实例添加指定数量的工作日数，返回当前实例|
| subWeekday | $value(1)| 当前实例减去指定数量的工作日数，返回当前实例|
| addWeekdays | $value| 同`addWeekday`|
| subWeekdays | $value| 同`subWeekday`|
| addWeek | $value(1)| 当前实例添加指定数量的星期数，返回当前实例|
| subWeek | $value(1)| 当前实例减去指定数量的星期数，返回当前实例|
| addWeeks | $value| 同`addWeek`|
| subWeeks | $value| 同`subWeek`|
| addMonth | $value(1)| 当前实例添加指定数量的月数，返回当前实例|
| subMonth | $value(1)| 当前实例减去指定数量的月数，返回当前实例|
| addMonths | $value  | 同`addMonth`|
| subMonths | $value  | 同`subMonth`|
| addMonthWithOverflow | $value(1)| 当前实例添加指定数量的月数（可溢出），返回当前实例|
| subMonthWithOverflow | $value(1)| 当前实例添加指定数量的月数（可溢出），返回当前实例|
| addMonthsWithOverflow | $value| 同`addMonthWithOverflow`|
| subMonthsWithOverflow | $value| 同`subMonthWithOverflow`|
| addMonthNoOverflow | $value(1)| 当前实例添加指定数量的月数（不可溢出），返回当前实例|
| subMonthNoOverflow | $value(1)| 当前实例添加指定数量的月数（不可溢出），返回当前实例|
| addMonthsNoOverflow | $value| 同`addMonthNoOverflow`|
| subMonthsNoOverflow | $value| 同`subMonthNoOverflow`|
| addQuarter | $value(1)| 当前实例添加指定数量的季度数，返回当前实例 |
| subQuarter | $value(1)| 当前实例减去指定数量的季度数，返回当前实例 |
| addQuarters | $value| 同`addQuarter`|
| subQuarters |$value | 同`subQuarter` |
| addYear | $value(1)| 当前实例添加指定数量的年数，返回当前实例|
| subYear | $value(1)| 当前实例减去指定数量的年数，返回当前实例|
| addYears |$value | 同`addYear`|
| subYears |$value | 同`subYear`|
| addCentury | $value(1)| 当前实例添加指定数量的世纪数，返回当前实例|
| subCentury | $value(1)| 当前实例减去指定数量的世纪数，返回当前实例|
| addCenturies | $value| 同`addCentury`|
| subCenturies | $value| 同`subCentury`|

### 用途：时间差值比较
| 方法名 | 参数 | 描述 |
|: --- :|: --- :|: --- :|
| diffInSeconds | Carbon $dt(null), $abs(true)| 获取指定`Carbon`对象与当前实例时间的秒数差，前者 - 后者； `abs`表示是否返回绝对值 |
| secondsSinceMidnight | | 获取当前实例时间的 0时0分0秒 与当前实例时间的秒差|
| secondsUntilEndOfDay| | 获取当前实例时间的 23时59分59秒 与当前实例时间的秒差 |
| diffInMinutes | Carbon $dt(null), $abs(true)| 获取指定`Carbon`对象与当前实例时间的分钟差, 取整 |
| diffInHours | Carbon $dt(null), $abs(true) | 获取指定`Carbon`对象与当前实例时间的小时差, 取整|
| diffInHoursFiltered | Closure $callback, Carbon $dt(null), $abs(true)| 获取指定`Carbon`对象与当前实例时间的(通过回调函数较验的)小时差, 取整|
| diffInDays | Carbon $dt(null), $abs(true)|获取指定`Carbon`对象与当前实例时间的天数差, 取整 |
| diffInDaysFiltered | Closure $callback, Carbon $dt(null), $abs(true)|获取指定`Carbon`对象与当前实例时间的(通过回调函数较验的)天数差, 取整 |
| diffInWeekdays | Carbon $dt(null), $abs(true)| 获取指定`Carbon`对象与当前实例时间的工作日数差, 取整|
| diffInWeekendDays | Carbon $dt(null), $abs(true)| 获取指定`Carbon`对象与当前实例时间的双休日数差, 取整|
| diffInWeeks |Carbon $dt(null), $abs(true) | 获取指定`Carbon`对象与当前实例时间的星期数差, 取整|
| diffInMonths | Carbon $dt(null), $abs(true)| 获取指定`Carbon`对象与当前实例时间的月数差, 取整|
| diffInYears | Carbon $dt(null), $abs(true)| 获取指定`Carbon`对象与当前实例时间的年数差, 取整|
| diffFiltered | CarbonInterval $ci， Closure $callback, Carbon $dt(null), $abs(true) | 获取指定`Carbon`对象与当前实例时间的(通过回调函数较验的)`$ci`差, 取整 |
| diffForHumans | Carbon $other(null), $absolute(false), $short(false) | 获取指定`Carbon`对象与当前实例时间的时间差，以便于人类阅读的格式呈现 |

### 用途：Getter & Setter
| 方法名 | 参数 | 描述 |
|: --- :|: --- :|: --- :|
| getDays（static） | | 获取`days of the week` |
| getWeekStartsAt（static） | | 获取一周的第一天 |
| setWeekStartsAt（static） | $day| 设置一周的第一天 |
| getWeekEndsAt（static） | | 获取一周的最后一天|
| setWeekEndsAt（static） | $day| 设置一周的最后一天|
| getWeekendDays（static） | | 获取双休日（数组）|
| setWeekendDays（static） | $days | 设置双休日 |
| getTranslator（static） | | 获取`translator`实例 |
| setTranslator（static） | TranslatorInterface $translator| 设置`translator`实例 |
| getLocale（static） | | 获取当前本地化语言 |
| setLocale（static） | $locale | 设置当前本地化语言 |
| timezone | $value | 设置时区 |
| tz | $value| 设置时区|
| setTimezone | $value| 设置时区 |
| setToStringFormat（static）| $format | 设置变量`$toStringFormat` |
| resetToStringFormat（static） | | 设置变量`$toStringFormat`为默认值 |
| setTestNow（static） | $testNow(null)| 设置变量`$testNow`，测试专用，初始化时的`$now` |
| getTestNow（static） | | 获取变量`$testNow` |
| hasTestNow（static） | | 判断`$testNow`是否为空|

### 用途：其它
| 方法名 | 参数 | 描述 |
|: --- :|: --- :|: --- :|
| setUtf8（static） | $utf8| 设置是否采用 utf8 编码方式 |
| getLastErrors（static） | | 获取无效时间的错误格式模板 |
| serialize | | 返回当前实例的序列化字符串 |
| hasRelativeKeywords（static） | $time | 判断字符串中是否有指定的关键字 |
| shouldOverflowMonths（static）| | 获取变量`$monthsOverflow` |
| useMonthsOverflow（static） | $monthsOverflow(true) | 设置变量`$monthsOverflow` |
| resetMonthsOverflow（static）| | 重置变量`$monthsOverflow`为 `true` |
| __get | $name | 魔术方法 |
| __isset | $name| 魔术方法 |
| __set | $name, $value | 魔术方法 |

## CarbonInterval
### 用途：生成`CarbonInterval`实例
| 方法名 | 参数 | 描述 |
|: --- :|: --- :|: --- :|
| __construct | $years(1), $months(null), $weeks(null), $days(null), $hours(null), $minutes(null), $seconds(null)| 创建`CarbonInterval`实例 |
| create（static）| $years(1), $months(null), $weeks(null), $days(null), $hours(null), $minutes(null), $seconds(null)| 创建`CarbonInterval`实例 |
| second(static) | $value | 创建 `CarbonInterval` 实例|
| seconds(static) | $value |  创建 `CarbonInterval` 实例|
| minute(static) | $value | 创建 `CarbonInterval` 实例 |
| minutes(static) | $value | 创建 `CarbonInterval` 实例 |
| hour(static) | $value | 创建 `CarbonInterval` 实例 |
| hours(static) | $value | 创建 `CarbonInterval` 实例 |
| day(static) | $value | 创建 `CarbonInterval` 实例 |
| days(static) | $value | 创建 `CarbonInterval` 实例 |
| dayz(static) | $value | 创建 `CarbonInterval` 实例 |
| week(static) | $value | 创建 `CarbonInterval` 实例 |
| weeks(static) | $value | 创建 `CarbonInterval` 实例 |
| month(static) | $value | 创建 `CarbonInterval` 实例 |
| months(static) | $value | 创建 `CarbonInterval` 实例 |
| year(static) | $value | 创建 `CarbonInterval` 实例 |
| years(static) | $value | 创建 `CarbonInterval` 实例 |
| instance(static) |  $value | 创建 `CarbonInterval` 实例 |

### 用途：本地化
| 方法名 | 参数 | 描述 |
|: --- :|: --- :|: --- :|
| translator（static） | | 初始化`translator`实例 |
| getTranslator（static | | 获取`translator`实例 |
| setTranslator（static） | TranslatorInterface $translator | 设置`translator`实例|
| getLocale（static） | | 获取当前本地化语言 |
| setLocale（static） | $locale | 设置当前本地化语言 |

### 用途：计算
| 方法名 | 参数 | 描述 |
|: --- :|: --- :|: --- :|
| add | DateInterval $interval | 将指定`DateInterval`的时间叠加到当前实例 |

### 用途：格式化
| 方法名 | 参数 | 描述 |
|: --- :|: --- :|: --- :|
| spec | | 获取规范的间隔描述字符串 |
| forHumans | | 获取便于人类阅读的间隔描述字符串 |
| __toString | | 同`forHumans`|

### 用途：其它
| 方法名 | 参数 | 描述 |
|: --- :|: --- :|: --- :|
| __get | | 魔术方法；可操作变量有`years`/`months`/`dayz`/`hours`/`minutes`/`seconds`/`weeks`/`daysExcludeWeeks`/`dayzExcludeWeeks`|
| __set | | 魔术方法；可操作变量有`years`/`months`/`dayz`/`hours`/`minutes`/`seconds`/`weeks`/`daysExcludeWeeks`/`dayzExcludeWeeks`|
| __call | | 魔术方法；可操作方法有`years`/`year`/`months`/`month`/`weeks`/`week`/`days`/`dayz`/`day`/`hours`/`hour`/`minutes`/`minute`/`seconds`/`second`|
| weeksAndDays | $weeks, $days | 为当前实例的`dayz`变量赋值为`($weeks * Carbon::DAYS_PER_WEEK) + $days`|
