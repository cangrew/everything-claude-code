---
description: "C patterns extending common rules"
globs: ["**/*.c", "**/*.h", "**/Makefile", "**/Makefile.*"]
alwaysApply: false
---
# C Patterns

> This file extends the common patterns rule with C specific content.

## Opaque Pointer (Information Hiding)

```c
/* mylib.h */
typedef struct MyLib MyLib;
MyLib* mylib_create(void);
void   mylib_destroy(MyLib* lib);

/* mylib.c */
struct MyLib { int internal_state; };
```

## Error Handling with `goto` Cleanup

```c
int process(const char* path) {
    FILE* f   = NULL;
    char* buf = NULL;
    int   rc  = -1;

    f = fopen(path, "r");
    if (!f) goto cleanup;

    buf = malloc(BUF_SIZE);
    if (!buf) goto cleanup;

    /* ... */
    rc = 0;
cleanup:
    free(buf);
    if (f) fclose(f);
    return rc;
}
```

## Vtable Polymorphism

```c
typedef struct {
    void (*draw)(void* self);
    void (*destroy)(void* self);
} ShapeVtable;

typedef struct { const ShapeVtable* vtable; } Shape;
```
