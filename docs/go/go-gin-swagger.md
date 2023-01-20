## Enable Authorize

in `main.go` Definition:
```
// @securityDefinitions.apikey ApiKeyAuth
// @in header
// @name X-api-key
```

in handler/controller:
```
// @Security ApiKeyAuth
func myHandler(c *gin.Context) {
    apikey := c.GetHeader("X-api-key")
}
```

!!! Reference

    Refer to [gin-swagger](https://github.com/swaggo/gin-swagger/issues/90) using [swag](https://github.com/swaggo/swag).