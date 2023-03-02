## Android Apple Login Flow

![apple login diagram](../img/mermaid-diagram-2023-02-28-144545.png)

## Redirect Server

Go Example:
```
func AppleLoginRedirect(c *gin.Context) {
	if c.Request.Method == "POST" {
		strBody, err := io.ReadAll(c.Request.Body)
		if err != nil {
			log.Fatal(err)
		}
		log.Println(string(strBody))

		c.Redirect(http.StatusFound, "coolish://callback?"+string(strBody))
	}
}

//router
r := gin.New()
r.Any("/applelogin/redirect", AppleLoginRedirect)
```