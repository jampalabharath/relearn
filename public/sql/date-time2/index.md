# Date & Time Formats

This guide demonstrates **all possible date parts, number formats, and culture-specific styles** available in SQL Server.  

---

## 1. Numeric Format Specifiers
```sql
SELECT 'N' AS FormatType, FORMAT(1234.56, 'N') AS FormattedValue
UNION ALL
SELECT 'P', FORMAT(1234.56, 'P')
UNION ALL
SELECT 'C', FORMAT(1234.56, 'C')
UNION ALL
SELECT 'E', FORMAT(1234.56, 'E')
UNION ALL
SELECT 'F', FORMAT(1234.56, 'F')
UNION ALL
SELECT 'N0', FORMAT(1234.56, 'N0')
UNION ALL
SELECT 'N1', FORMAT(1234.56, 'N1')
UNION ALL
SELECT 'N2', FORMAT(1234.56, 'N2')
UNION ALL
SELECT 'N_de-DE', FORMAT(1234.56, 'N', 'de-DE')
UNION ALL
SELECT 'N_en-US', FORMAT(1234.56, 'N', 'en-US');
```

## 2. Date Format Specifiers
```sql
SELECT 'D' AS FormatType, FORMAT(GETDATE(), 'D') AS FormattedValue, 'Full date pattern' AS Description
UNION ALL SELECT 'd', FORMAT(GETDATE(), 'd'), 'Short date pattern'
UNION ALL SELECT 'dd', FORMAT(GETDATE(), 'dd'), 'Day of month with leading zero'
UNION ALL SELECT 'ddd', FORMAT(GETDATE(), 'ddd'), 'Abbreviated name of day'
UNION ALL SELECT 'dddd', FORMAT(GETDATE(), 'dddd'), 'Full name of day'
UNION ALL SELECT 'M', FORMAT(GETDATE(), 'M'), 'Month without leading zero'
UNION ALL SELECT 'MM', FORMAT(GETDATE(), 'MM'), 'Month with leading zero'
UNION ALL SELECT 'MMM', FORMAT(GETDATE(), 'MMM'), 'Abbreviated name of month'
UNION ALL SELECT 'MMMM', FORMAT(GETDATE(), 'MMMM'), 'Full name of month'
UNION ALL SELECT 'yy', FORMAT(GETDATE(), 'yy'), 'Two-digit year'
UNION ALL SELECT 'yyyy', FORMAT(GETDATE(), 'yyyy'), 'Four-digit year'
UNION ALL SELECT 'hh', FORMAT(GETDATE(), 'hh'), '12-hour clock with leading zero'
UNION ALL SELECT 'HH', FORMAT(GETDATE(), 'HH'), '24-hour clock with leading zero'
UNION ALL SELECT 'm', FORMAT(GETDATE(), 'm'), 'Minutes without leading zero'
UNION ALL SELECT 'mm', FORMAT(GETDATE(), 'mm'), 'Minutes with leading zero'
UNION ALL SELECT 's', FORMAT(GETDATE(), 's'), 'Seconds without leading zero'
UNION ALL SELECT 'ss', FORMAT(GETDATE(), 'ss'), 'Seconds with leading zero'
UNION ALL SELECT 'f', FORMAT(GETDATE(), 'f'), 'Tenths of a second'
UNION ALL SELECT 'ff', FORMAT(GETDATE(), 'ff'), 'Hundredths of a second'
UNION ALL SELECT 'fff', FORMAT(GETDATE(), 'fff'), 'Milliseconds'
UNION ALL SELECT 'T', FORMAT(GETDATE(), 'T'), 'Full AM/PM designator'
UNION ALL SELECT 't', FORMAT(GETDATE(), 't'), 'Single char AM/PM designator'
UNION ALL SELECT 'tt', FORMAT(GETDATE(), 'tt'), 'Two char AM/PM designator';
```
## 3. DatePart / DateName / DateTrunc Comparisons
```sql
SELECT 'Year' AS DatePart, DATEPART(year, GETDATE()), DATENAME(year, GETDATE()), DATETRUNC(year, GETDATE())
UNION ALL SELECT 'Quarter', DATEPART(quarter, GETDATE()), DATENAME(quarter, GETDATE()), DATETRUNC(quarter, GETDATE())
UNION ALL SELECT 'Month', DATEPART(month, GETDATE()), DATENAME(month, GETDATE()), DATETRUNC(month, GETDATE())
UNION ALL SELECT 'DayOfYear', DATEPART(dayofyear, GETDATE()), DATENAME(dayofyear, GETDATE()), DATETRUNC(dayofyear, GETDATE())
UNION ALL SELECT 'Day', DATEPART(day, GETDATE()), DATENAME(day, GETDATE()), DATETRUNC(day, GETDATE())
UNION ALL SELECT 'Week', DATEPART(week, GETDATE()), DATENAME(week, GETDATE()), DATETRUNC(week, GETDATE())
UNION ALL SELECT 'Weekday', DATEPART(weekday, GETDATE()), DATENAME(weekday, GETDATE()), NULL
UNION ALL SELECT 'Hour', DATEPART(hour, GETDATE()), DATENAME(hour, GETDATE()), DATETRUNC(hour, GETDATE())
UNION ALL SELECT 'Minute', DATEPART(minute, GETDATE()), DATENAME(minute, GETDATE()), DATETRUNC(minute, GETDATE())
UNION ALL SELECT 'Second', DATEPART(second, GETDATE()), DATENAME(second, GETDATE()), DATETRUNC(second, GETDATE())
UNION ALL SELECT 'Millisecond', DATEPART(millisecond, GETDATE()), DATENAME(millisecond, GETDATE()), DATETRUNC(millisecond, GETDATE())
UNION ALL SELECT 'Microsecond', DATEPART(microsecond, GETDATE()), DATENAME(microsecond, GETDATE()), NULL
UNION ALL SELECT 'Nanosecond', DATEPART(nanosecond, GETDATE()), DATENAME(nanosecond, GETDATE()), NULL
UNION ALL SELECT 'ISOWeek', DATEPART(iso_week, GETDATE()), DATENAME(iso_week, GETDATE()), DATETRUNC(iso_week, GETDATE());
```

## 4. Culture Code Formatting
```sql
SELECT 'en-US' AS CultureCode, FORMAT(1234567.89, 'N', 'en-US'), FORMAT(GETDATE(), 'D', 'en-US')
UNION ALL SELECT 'en-GB', FORMAT(1234567.89, 'N', 'en-GB'), FORMAT(GETDATE(), 'D', 'en-GB')
UNION ALL SELECT 'fr-FR', FORMAT(1234567.89, 'N', 'fr-FR'), FORMAT(GETDATE(), 'D', 'fr-FR')
UNION ALL SELECT 'de-DE', FORMAT(1234567.89, 'N', 'de-DE'), FORMAT(GETDATE(), 'D', 'de-DE')
UNION ALL SELECT 'es-ES', FORMAT(1234567.89, 'N', 'es-ES'), FORMAT(GETDATE(), 'D', 'es-ES')
UNION ALL SELECT 'zh-CN', FORMAT(1234567.89, 'N', 'zh-CN'), FORMAT(GETDATE(), 'D', 'zh-CN')
UNION ALL SELECT 'ja-JP', FORMAT(1234567.89, 'N', 'ja-JP'), FORMAT(GETDATE(), 'D', 'ja-JP')
UNION ALL SELECT 'ko-KR', FORMAT(1234567.89, 'N', 'ko-KR'), FORMAT(GETDATE(), 'D', 'ko-KR')
UNION ALL SELECT 'pt-BR', FORMAT(1234567.89, 'N', 'pt-BR'), FORMAT(GETDATE(), 'D', 'pt-BR')
UNION ALL SELECT 'it-IT', FORMAT(1234567.89, 'N', 'it-IT'), FORMAT(GETDATE(), 'D', 'it-IT')
UNION ALL SELECT 'nl-NL', FORMAT(1234567.89, 'N', 'nl-NL'), FORMAT(GETDATE(), 'D', 'nl-NL')
UNION ALL SELECT 'ru-RU', FORMAT(1234567.89, 'N', 'ru-RU'), FORMAT(GETDATE(), 'D', 'ru-RU')
UNION ALL SELECT 'ar-SA', FORMAT(1234567.89, 'N', 'ar-SA'), FORMAT(GETDATE(), 'D', 'ar-SA')
UNION ALL SELECT 'el-GR', FORMAT(1234567.89, 'N', 'el-GR'), FORMAT(GETDATE(), 'D', 'el-GR')
UNION ALL SELECT 'tr-TR', FORMAT(1234567.89, 'N', 'tr-TR'), FORMAT(GETDATE(), 'D', 'tr-TR')
UNION ALL SELECT 'he-IL', FORMAT(1234567.89, 'N', 'he-IL'), FORMAT(GETDATE(), 'D', 'he-IL')
UNION ALL SELECT 'hi-IN', FORMAT(1234567.89, 'N', 'hi-IN'), FORMAT(GETDATE(), 'D', 'hi-IN');
```