# Model de dades

## Model Entitat Relació E/R

```mermaid
erDiagram
    User ||--o{ Post : id
    User {
        string username
        string password
    }
    Post {
        date created
        string title
        string body
    }
```

| End-Point | Methods | Descripció |
| ----------- | ----------- | ---------|
| /Register | Get |
| /Register | Post |
| /Login    | Get |
| /Login    | Post |
| /Logout   | Get |
| /Create   | Get |
| /Create   | Post |
| /<int:id>/update | Get |
| /<int:id>/update | Post |
| /<int:id>/delete | Post |
| / | Get |
| /hello | Get |