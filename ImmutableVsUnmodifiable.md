---
tags: Java
title: Immutable vs Unmodifiable
description: Use `{%hackmd BkVfcTxlQ %}` syntax to include this theme.
---
# Immutable vs Unmodifiable

## Definitions
From the Collections Framework Overview:

>:small_red_triangle_down: Collections that do not support modification operations (such as add, remove and clear) are referred to as unmodifiable. Collections that are not unmodifiable are modifiable.
>:small_red_triangle_down: Collections that additionally guarantee that no change in the Collection object will be visible are referred to as immutable. Collections that are not immutable are mutable.

## Example

```java=
@Test
public void testList() {

    List<String> modifiableList = new ArrayList<String>();
    modifiableList.add("a");

    System.out.println("modifiableList:"+modifiableList);
    System.out.println("--");


    //unModifiableList

    assertEquals(1, modifiableList.size());

    List<String> unModifiableList=Collections.unmodifiableList(
                                        modifiableList);

    modifiableList.add("b");

    boolean exceptionThrown=false;
    try {
        unModifiableList.add("b");
        fail("add supported for unModifiableList!!");
    } catch (UnsupportedOperationException e) {
        exceptionThrown=true;
        System.out.println("unModifiableList.add() not supported");
    }
    assertTrue(exceptionThrown);

    System.out.println("modifiableList:"+modifiableList);
    System.out.println("unModifiableList:"+unModifiableList);

    assertEquals(2, modifiableList.size());
    assertEquals(2, unModifiableList.size());
            System.out.println("--");



            //immutableList


    List<String> immutableList=Collections.unmodifiableList(
                            new ArrayList<String>(modifiableList));

    modifiableList.add("c");

    exceptionThrown=false;
    try {
        immutableList.add("c");
        fail("add supported for immutableList!!");
    } catch (UnsupportedOperationException e) {
        exceptionThrown=true;
        System.out.println("immutableList.add() not supported");
    }
    assertTrue(exceptionThrown);


    System.out.println("modifiableList:"+modifiableList);
    System.out.println("unModifiableList:"+unModifiableList);
    System.out.println("immutableList:"+immutableList);
    System.out.println("--");

    assertEquals(3, modifiableList.size());
    assertEquals(3, unModifiableList.size());
    assertEquals(2, immutableList.size());

}
```
```java=
modifiableList:[a]
--
unModifiableList.add() not supported
modifiableList:[a, b]
unModifiableList:[a, b]
--
immutableList.add() not supported
modifiableList:[a, b, c]
unModifiableList:[a, b, c]
immutableList:[a, b]
--
```

## References
1. [Immutable vs Unmodifiable collection](https://stackoverflow.com/questions/8892350/immutable-vs-unmodifiable-collection)

---
<style>

html, body, .ui-content {
    background-color: #333;
    color: #ddd;
}

body > .ui-infobar {
    display: none;
}

.ui-view-area > .ui-infobar {
    display: block;
}

.markdown-body h1,
.markdown-body h2,
.markdown-body h3,
.markdown-body h4,
.markdown-body h5,
.markdown-body h6 {
    color: #ddd;
}

.markdown-body h1,
.markdown-body h2 {
    border-bottom-color: #ffffff69;
}

.markdown-body h1 .octicon-link,
.markdown-body h2 .octicon-link,
.markdown-body h3 .octicon-link,
.markdown-body h4 .octicon-link,
.markdown-body h5 .octicon-link,
.markdown-body h6 .octicon-link {
    color: #fff;
}

.markdown-body img {
    background-color: transparent;
}

.ui-toc-dropdown .nav>.active:focus>a, .ui-toc-dropdown .nav>.active:hover>a, .ui-toc-dropdown .nav>.active>a {
    color: white;
    border-left: 2px solid white;
}

.expand-toggle:hover, 
.expand-toggle:focus, 
.back-to-top:hover, 
.back-to-top:focus, 
.go-to-bottom:hover, 
.go-to-bottom:focus {
    color: white;
}


.ui-toc-dropdown {
    background-color: #333;
}

.ui-toc-label.btn {
    background-color: #191919;
    color: white;
}

.ui-toc-dropdown .nav>li>a:focus, 
.ui-toc-dropdown .nav>li>a:hover {
    color: white;
    border-left: 1px solid white;
}

.markdown-body blockquote {
    color: #bcbcbc;
}

.markdown-body table tr {
    background-color: #5f5f5f;
}

.markdown-body table tr:nth-child(2n) {
    background-color: #4f4f4f;
}

.markdown-body code,
.markdown-body tt {
    color: #eee;
    background-color: rgba(230, 230, 230, 0.36);
}

a,
.open-files-container li.selected a {
    color: #5EB7E0;
}


</style>