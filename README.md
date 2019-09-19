# idea-of-test-visuals

This is the idea paper for testing what we can see from screen. For example, web page or game client including ***animation effect***.

# Motivation

[PhantomCSS](https://github.com/HuddleEng/PhantomCSS) test web page's visuals using screenshot, comparing pixel by pixel. But it cannot test animations.
We can save all frames. Than developer can confirm the pixels in saved frame is good or bad by their eyes.

After changing codes, run test and capture all frames.
Compare new frames with previous test's frames which passed test before.
If there is no problem, visual test is pass. If there is difference, developer can check it and confirm or fix the codes.

But there is problem. How to make sure the frame is always the same over the test? To make sure that, we try to control which would make frame different over the test.

# Main Idea; Control which changes frame over the test!

1. Control rendering frames
2. Control time
3. Control I/O

# Test screen except animation effect

It is easier to test web page which has no animation effect. Just compare screenshot and check pixels. There is already well made framework like [PhantomCSS](https://github.com/HuddleEng/PhantomCSS).

# Test screen including animation effect

It's more difficult to test which has animation effects. Because,
1. Animation effect is moving.
2. Animation effect depends on time.
3. Time is not controllable, Because of
  1. Async operation.
  2. Resource(Processor) Scheduling


It doesn't mean that it is impossible. It is just difficult. This paper presents new idea to test screen with animation effect.

# 1. Control rendering frames in test code

Basically the rendering engine usually iterates loop and renders over and over.

We don't have to run loops during test. We can invoke render function by ourselves.

Here is pseudo code to control rendering frames in test code.

``` typescript
class engine {
  /// render {frames} times
  public RenderFrames(frames: number) {
    for (var i = 0; i < frames; i += 1) {
      Update(deltaTime);
      Render(deltaTime);
      var frame = CaptureFrame();
      CheckFrameIsGood(frame); // Compare with saved, previously test passed frame.
    }
  }
}

...

test(("click play button") => {
  engine.RenderFrames(20); // render 20 frames

  ClickPlayButton();

  engine.RenderFrames(10);
});
```

It seems good and will work very well if it is satisfying 2 requirement below,
1. deltaTime in `RenderFrames(frames)` is always same over the test
  - We need to ***Control Time***
2. it reacts immediately when click play button.
  - We need to ***Control I/O***

The next parts of this paper will discuss ways to satisfy the requirements.

# 2. Control time in test code

When we calculate deltaTime for rendering or updating, we just calculate `(current time) - (time of last rendering)`.

It's impossible to make the same deltaTime using above calculation. Because computer(or OS) has its own scheduler and we cannot control it in user mode.

Simply, ***we can set deltaTime as constance number.*** for example, 33ms. than we can test without thinking timing issue!

We have to make sure all animations animate using deltaTime of render method or update method.


``` typescript
const deltaTime = 33; // 33ms always.

class engine {
  /// render {frames} times
  public RenderFrames(frames: number) {
    for (var i = 0; i < frames; i += 1) {
      Update(deltaTime);
      Render(deltaTime);
      var frame = CaptureFrame();
      CheckFrameIsGood(frame); // Compare with saved, previously test passed frame.
    }
  }
}

...

test(("click play button") => {
  // <- 0 ms

  engine.RenderFrames(20); // render 20 frames

  // <- 660ms (deltaTime 33 * 20)

  ClickPlayButton();

  engine.RenderFrames(10);

  // <- 990ms
});
```

In above test code, we can make sure how many seconds exactly passed between methods.

But if clicking play button doesn't react immediately, we should wait for it.

# 3. Control I/O in test code

// Need to sleep. I will write this part tomorrow.
// Idea : Mocking every I/O for test code like unit test.
// Because we want to test screen, not End-to-end test. right? right.