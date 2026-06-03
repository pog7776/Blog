---
layout: post
title: My Unity event coroutine adventure
date: 2026-06-03 14:30
categories: [Unity]
tags: [Unity, Coroutine, Events]
description: Lets explore some interesting `Start()` method behaviour.
image: /assets/img/blog/2026-06-03-unityeventcoroutines/banner.png
toc: false
---

## Event Function Coroutines

The other day I discovered that some Unity event functions can be defined as a coroutine by simply setting their
return type to `IEnumerator`.
<br>
[Source](https://docs.unity3d.com/6000.4/Documentation/ScriptReference/Coroutine.html)

**Example**
```csharp
private IEnumerator Start()
{
    yield return Task();
}

private IEnumerator Task()
{
    yield return new WaitForSeconds(10f);
}
```

---

## The Weirdness

This is pretty handy if you want full control over the flow of the script but can lead to some
unexpected behaviour if you're not paying attention.

Lets have a look at this example I put together:

```csharp
public class StartIEnumeratorTest : MonoBehaviour
{
    private bool _isStart;

    private IEnumerator Start()
    {
        while(true)
        {
            if(_isStart)
            {
                Debug.LogWarning("Double Start");
            }

            Debug.Log("Start loop");

            _isStart = true;
            yield return null;
        }
    }

    private void Update()
    {
        if(!_isStart)
        {
            Debug.LogWarning("Double Update");
        }

        Debug.Log("Update loop");

        _isStart = false;
    }
}
```

> The relevant Unity execution order is as follows:
> 
> 1. `Start`
> 2. `Update`
> 3. `Coroutine`
>
> [Source](https://docs.unity3d.com/6000.2/Documentation/Manual/execution-order.html)

So stepping through the first few frames of this script we'll get:

|                | Frame 1                      | Frame 2                        | Frame 3                        |
|----------------|------------------------------|--------------------------------|--------------------------------|
| **Call**       | `Start` → `Update`           | `Update` → `Start Coroutine`   | `Update` → `Start Coroutine`   |
| **Log output** | `Start loop` → `Update loop` | `Double Update` → `Start loop` | `Update loop` → `Start loop`   |

Notice that double update we get between the first and second frame?

In the first frame the `Start` contents will run before `Update` as expected.
<br>
After the second frame `Update` will always execute before the `Start` coroutine.

This usually won't cause any trouble,
but it's handy to be aware of this if you're working with event function coroutines.

## Conclusion

Super useful functionality, just be careful!

Thanks for reading my first actual blog post! 
<br>
Lets see if I can improve my writing and the readability of the content.
