# Writers

`http.ResponseWriter` parameter provides methods for assembling a HTTP response and sending it to the user, e.g. WriteHeader, Write.

```go
func home(w http.ResponseWriter, r *http.Request) {
	w.Write([]byte("Hello from Snippet box"))
}

func main() {
	fmt.Println("hi")

	mux := http.NewServeMux()
	mux.HandleFunc("/", home)
	log.Print("starting server on 4000")

	err := http.ListenAndServe(":4000", mux)
	log.Fatal(err)

}
```
