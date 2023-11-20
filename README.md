# Learning Go Fiber

Sebenarnya banyak sekali Web Framework yang bisa kita gunakan untuk memudahkan
kita membuat aplikasi Web dengan Golang. di Golang sendiri salah satu Web
Framework yang populer saat ini yaitu Fiber atau Go Fiber. Fiber adalah Web
Framework untuk Golang yang terinspirasi dari ExpressJS, oleh karena itu cara
penggunaannya sangat mudah dan sederhana.

## Membuat Project

Buat sebuah folder dengan nama misalnya **_GOLANG_FIBER_**, setelah itu lakukan
inisiasi project golang dengan mengetikkan di terminal.

```bash
go mod init golang-fiber
```

## Menambahkan Depedensi

Ada beberapa depedensi yang kita perlukan dalam project ini seperti testify
untuk membuat unit test dan lainnya. Untuk menambahkan depedensi, caranya
ketikkan di terminal.

1. Testify

```bash
go get github.com/stretchr/testify
```

2. Go Fiber

```bash
go get github.com/gofiber/fiber/v2
```

## Memulai Project

Setelah menambahkan depedensi, selanjutnya buka project menggunakan text editor
atau IDE favorit teman-teman. Kemudian buat sebuah file dengan nama _main.go_

- file _main.go_

```go
package main

import "github.com/gofiber/fiber/v2"

func main() {
 app := fiber.New()

 err := app.Listen("localhost:3000")
 if err != nil {
  panic(err)
 }
}
```

## Configuration

Saat membuat fiber.App menggunakan fiber.New() terdapat parameter fiber.Config
yang bisa kita gunakan. Ada banyak sekali konfigurasi yang bisa kita ubah,
contohnya mengubah konfigurasi timeout.

```go
package main

import (
 "time"

 "github.com/gofiber/fiber/v2"
)

func main() {
 app := fiber.New(fiber.Config{
  IdleTimeout:  time.Second * 5,
  ReadTimeout:  time.Second * 5,
  WriteTimeout: time.Second * 5,
 })

 err := app.Listen("localhost:3000")
 if err != nil {
  panic(err)
 }
}
```

## Routing

Sekarang kita bahas tentang Routing. di Fiber untuk membuat Routing sudah
disediakan semua Method di fiber.App yang sesuai dengan HTTPMethod. Parameternya
membutuhkan 2, yaitu path nya dan juga fiber.Handler. Kode: Routing Method

```go
// HTTP methods
func (app *App) Get(path string, handlers ...Handler) Router
func (app *App) Head(path string, handlers ...Handler) Router
func (app *App) Post(path string, handlers ...Handler) Router
func (app *App) Put(path string, handlers ...Handler) Router
func (app *App) Delete(path string, handlers ...Handler) Router
func (app *App) Connect(path string, handlers ...Handler) Router
func (app *App) Options(path string, handlers ...Handler) Router
func (app *App) Trace(path string, handlers ...Handler) Router
func (app *App) Patch(path string, handlers ...Handler) Router
```

Ok, sekarang kita coba buat. Buka file _main.go_

- file _main.go_

```go
package main

import (
        "time"

        "github.com/gofiber/fiber/v2"
)

func main() {
    app := fiber.New(fiber.Config{
        IdleTimeout:  time.Second * 5,
        ReadTimeout:  time.Second * 5,
        WriteTimeout: time.Second * 5,
    })

    // Routing
        app.Get("/", func(ctx *fiber.Ctx) error {
        return ctx.SendString("Hello World")
    })

        err := app.Listen("localhost:3000")
    if err != nil {
        panic(err)
    }
}
```

Sekarang kita akan mengimplementasikan unit test. Buat sebuah file dengan nama
_fiber_test.go_

- file _fiber_test.go_

```go
package main

import (
 "io"
 "net/http/httptest"
 "testing"

 "github.com/gofiber/fiber/v2"
 "github.com/stretchr/testify/assert"
)

func TestRoutingHelloWorld(t *testing.T) {
 app := fiber.New()
 app.Get("/", func(ctx *fiber.Ctx) error {
  return ctx.SendString("Hello World")
 })

 request := httptest.NewRequest("GET", "/", nil)
 response, err := app.Test(request)
 assert.Nil(t, err)
 assert.Equal(t, 200, response.StatusCode)

 bytes, err := io.ReadAll(response.Body)
 assert.Nil(t, err)
 assert.Equal(t, "Hello World", string(bytes))
}
```

## Context di Fiber (Ctx)

Saat kita membuat Handler di Fiber Routing, kita hanya cukup menggunakan
parameter fiber.Ctx. Ctx ini merupakan representasi dari Request dan Response di
Fiber. Oleh karena itu, kita bisa mendapatkan informasi HTTP Request, dan juga
bisa membuat HTTP Response menggunakan fiber.Ctx.

- Buka kembali file _fiber_test.go_

```go
func TestCtx(t *testing.T) {
    app.Get("/hello", func(ctx *fiber.Ctx) error {
        name := ctx.Query("name", "Guest")
        return ctx.SendString("Hello " + name)
    })

        request := httptest.NewRequest("GET", "/hello?name=Mazufik", nil)
    response, err := app.Test(request)
    assert.Nil(t, err)
    assert.Equal(t, 200, response.StatusCode)

        bytes, err := io.ReadAll(response.Body)
    assert.Nil(t, err)
    assert.Equal(t, "Hello Mazufik", string(bytes))

        request = httptest.NewRequest("GET", "/hello", nil)
    response, err = app.Test(request)
    assert.Nil(t, err)
    assert.Equal(t, 200, response.StatusCode)

        bytes, err = io.ReadAll(response.Body)
    assert.Nil(t, err)
    assert.Equal(t, "Hello Guest", string(bytes))
}
```

## HTTP Request

Representasi dari HTTP Request di FIber adalah Ctx. Untuk mengambil informasi
dari HTTP Request, kita bisa menggunakan banyak sekali Method yang terdapat di
Ctx.

- Buka kembali file _fiber_test.go_

```go
func TestHttpRequest(t *testing.T) {
 app.Get("/request", func(ctx *fiber.Ctx) error {
  first := ctx.Get("firstname")
  last := ctx.Cookies("lastname")
  return ctx.SendString("Hello " + first + " " + last)
 })

 request := httptest.NewRequest("GET", "/request", nil)
 request.Header.Set("firstname", "Mazufik")
 request.AddCookie(&http.Cookie{Name: "lastname", Value: "Rahman"})
 response, err := app.Test(request)
 assert.Nil(t, err)
 assert.Equal(t, 200, response.StatusCode)

 bytes, err := io.ReadAll(response.Body)
 assert.Nil(t, err)
 assert.Equal(t, "Hello Mazufik Rahman", string(bytes))
}
```

## Route Parameter

Fiber mendukung Route Parameter, dimana kita bisa menambahkan parameter di path
URL. Ini sangat cocok ketika kita membuat RESTful API, yang butuh data dikirim
via path URL. Saat membuat Route Parameter, kita perlu memberi nama, dan di Ctx,
kita bisa mengambil seluruh data menggunakan method AllParams(), atau
menggunakan method Params(nama).

- Buka kembali file _fiber_test.go_

```go
func TestRouteParameter(t *testing.T) {
 app.Get("/users/:userId/orders/:orderId", func(ctx *fiber.Ctx) error {
  userId := ctx.Params("userId")
  orderId := ctx.Params("orderId")
  return ctx.SendString("Get Order " + orderId + " From User " + userId)
 })

 request := httptest.NewRequest("GET", "/users/mazufik/orders/12", nil)
 response, err := app.Test(request)
 assert.Nil(t, err)
 assert.Equal(t, 200, response.StatusCode)

 bytes, err := io.ReadAll(response.Body)
 assert.Nil(t, err)
 assert.Equal(t, "Get Order 12 From User mazufik", string(bytes))
}
```

## Request Form

Ketika kita mengirim data menggunakan HTTP Form, kita bisa menggunakan method
FormValue(name) pada Ctx untuk mendapatkan data yang dikirim.

- Buka kembali file _fiber_test.go_

```go
func TestFormRequest(t *testing.T) {
 app.Post("/hello", func(ctx *fiber.Ctx) error {
  name := ctx.FormValue("name")
  return ctx.SendString("Hello " + name)
 })

 body := strings.NewReader("name=Mazufik")
 request := httptest.NewRequest("POST", "/hello", body)
 request.Header.Set("Content-Type", "application/x-www-form-urlencoded")
 response, err := app.Test(request)
 assert.Nil(t, err)
 assert.Equal(t, 200, response.StatusCode)

 bytes, err := io.ReadAll(response.Body)
 assert.Nil(t, err)
 assert.Equal(t, "Hello Mazufik", string(bytes))
}
```
