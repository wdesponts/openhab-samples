# Please share any XSL transforms you created here.

Yahoo Weather - temperature
```
<?xml version="1.0"?>
<xsl:stylesheet 
        xmlns:xsl="http://www.w3.org/1999/XSL/Transform"
        xmlns:yweather="http://xml.weather.yahoo.com/ns/rss/1.0" version="1.0">

        <xsl:output indent="yes" method="xml" encoding="UTF-8" omit-xml-declaration="yes" />

        <xsl:template match="/">
                <xsl:value-of select="//item/yweather:condition/@temp" /> 
        </xsl:template>


</xsl:stylesheet>
```

Yahoo Weather - feels like temperature
```
<?xml version="1.0"?>
<xsl:stylesheet 
        xmlns:xsl="http://www.w3.org/1999/XSL/Transform"
        xmlns:yweather="http://xml.weather.yahoo.com/ns/rss/1.0" version="1.0">

        <xsl:output indent="yes" method="xml" encoding="UTF-8" omit-xml-declaration="yes" />

        <xsl:template match="/">
                <xsl:value-of select="//yweather:wind/@chill" /> 
        </xsl:template>


</xsl:stylesheet>
```

Yahoo Weather - humidity
```
<?xml version="1.0"?>
<xsl:stylesheet 
        xmlns:xsl="http://www.w3.org/1999/XSL/Transform"
        xmlns:yweather="http://xml.weather.yahoo.com/ns/rss/1.0" version="1.0">

        <xsl:output indent="yes" method="xml" encoding="UTF-8" omit-xml-declaration="yes" />

        <xsl:template match="/">
                <xsl:value-of select="//yweather:atmosphere/@humidity" />
        </xsl:template>

</xsl:stylesheet>

```

Yahoo Weather - sunrise
```
<?xml version="1.0"?>
<xsl:stylesheet 
        xmlns:xsl="http://www.w3.org/1999/XSL/Transform"
        xmlns:yweather="http://xml.weather.yahoo.com/ns/rss/1.0" version="1.0">

        <xsl:output indent="yes" method="xml" encoding="UTF-8" omit-xml-declaration="yes" />

        <xsl:template match="/">
                <xsl:value-of select="//yweather:astronomy/@sunrise" />
        </xsl:template>

</xsl:stylesheet>
```

Yahoo Weather - sunset
```
<?xml version="1.0"?>
<xsl:stylesheet 
        xmlns:xsl="http://www.w3.org/1999/XSL/Transform"
        xmlns:yweather="http://xml.weather.yahoo.com/ns/rss/1.0" version="1.0">

        <xsl:output indent="yes" method="xml" encoding="UTF-8" omit-xml-declaration="yes" />

        <xsl:template match="/">
                <xsl:value-of select="//yweather:astronomy/@sunset" />
        </xsl:template>

</xsl:stylesheet>

```

Yahoo Weather - weather text
```
<?xml version="1.0"?>
<xsl:stylesheet 
        xmlns:xsl="http://www.w3.org/1999/XSL/Transform"
        xmlns:yweather="http://xml.weather.yahoo.com/ns/rss/1.0" version="1.0">

        <xsl:output indent="yes" method="xml" encoding="UTF-8" omit-xml-declaration="yes" />

        <xsl:template match="/">
                <xsl:value-of select="//item/yweather:condition/@text" />
        </xsl:template>

</xsl:stylesheet>

```

Yahoo Weather - wind speed
```
<?xml version="1.0"?>
<xsl:stylesheet 
        xmlns:xsl="http://www.w3.org/1999/XSL/Transform"
        xmlns:yweather="http://xml.weather.yahoo.com/ns/rss/1.0" version="1.0">

        <xsl:output indent="yes" method="xml" encoding="UTF-8" omit-xml-declaration="yes" />

        <xsl:template match="/">
                <xsl:value-of select="//yweather:wind/@speed" />
        </xsl:template>

</xsl:stylesheet>
```

Yahoo Weather - forecast tomorrow
```
<?xml version="1.0"?>
<xsl:stylesheet 
        xmlns:xsl="http://www.w3.org/1999/XSL/Transform"
        xmlns:yweather="http://xml.weather.yahoo.com/ns/rss/1.0" version="1.0">

        <xsl:output indent="yes" method="text" encoding="UTF-8" omit-xml-declaration="yes" />

        <xsl:template match="/">
                <xsl:value-of select="//item/yweather:forecast[1]/@text" />
                <xsl:text>, Low: </xsl:text>
                <xsl:value-of select="//item/yweather:forecast[1]/@low" /> 
                <xsl:text>°C, High: </xsl:text>
                <xsl:value-of select="//item/yweather:forecast[1]/@high" /> 
                <xsl:text>°C</xsl:text>
        </xsl:template>


</xsl:stylesheet>
```

Yahoo Weather - forecast day after tomorrow
```
<?xml version="1.0"?>
<xsl:stylesheet 
        xmlns:xsl="http://www.w3.org/1999/XSL/Transform"
        xmlns:yweather="http://xml.weather.yahoo.com/ns/rss/1.0" version="1.0">

        <xsl:output indent="yes" method="text" encoding="UTF-8" omit-xml-declaration="yes" />

        <xsl:template match="/">
                <xsl:value-of select="//item/yweather:forecast[2]/@text" />
                <xsl:text>, Low: </xsl:text>
                <xsl:value-of select="//item/yweather:forecast[2]/@low" /> 
                <xsl:text>°C, High: </xsl:text>
                <xsl:value-of select="//item/yweather:forecast[2]/@high" /> 
                <xsl:text>°C</xsl:text>
        </xsl:template>


</xsl:stylesheet>
```