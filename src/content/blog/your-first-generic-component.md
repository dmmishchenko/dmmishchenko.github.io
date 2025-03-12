---
title: "Building your first generic Angular component"
description: "This article helps you to understand how to build your generic components on Angular"
pubDate: "Mar 01 2025"
heroImage: "/blog-placeholder-3.jpg"
---

- Why you should not build your generic component
  - it's time consuming and maybe you don't have enough capacity for this
  - there are a lot of open source analogues that can be used, don't need to reinvent the wheel again. I suggest you to make a proper research. It could save you a lot of efforts.
  - This component tries to cover a lot of responsibilites at once. So it becomes more and more generic. That thing often happens when people blindly follows DRY rule. It's better to split it into separate specific components. It's okay to repeat some code.
- What you should consider if you want to build it(don't need to do it if you are only user of this component)
  - backward compatibility to not break anything, at some point after your component will be in use you need to preserve existing contract, so any changes should happen only inside your component, but contract will remain the same. You can ignore this while you're doing mvp version. If you need to apply some breaking changes think about migration script or schematic that can be applied by your users. It will save a lot of your and their time.
  - try to move faster in a small increments, it helps to get more feedback and not to stuck.
  - please cover your code with tests - it at least should be unit tests to cover how your component react and treat all inputs, does it emit all expected events or outputs
  - if you have capacity or motivation you can write some basic e2e tests to cover some main scenarious of your component. For example for table you can cover pagination, sorting, search actions. You should cover it as it helps you to be sure that all staples of your component work properly after any change.
  - think about a demo page: stylus with configurable options or any page that you can hide under dev flag in your app and share with other developers as a playground
  - be ready for silly, dumb and repetative questions from other developers
  - discuss support capacity with your manager
  - try not to involve a lot of people in discussion in the very beginning, as it will slow you down, but gather feedback after release from you users(other developers)
  - be ready to do small sessions with silly, dumb and repetative questions - it will be very obvious for you, but it happens because you're(probably working a lot with this component unlike other developers)
  - try to cover all main browsers and devices if you have spare capacity. Trust me agenda "we are using chrome" will change very soon for you
  - if you don't have any capacity or you have any other restrictions - discuss this with your managment. You can either mark somehow that this device or browser not very well supported or simply disable your component usage which is very bad idea.
  - learn how to publish changes in a proper and clear way so it will be seen by your users, you will need to learn how to write changelogs, update your demo page/playground, show some code chunks
  - try to cover your generic component with types and interfaces. It should be clear for your users what type this input expects and what type that output event has
  - I suggest you to utilize last stable Angular features as Signals, especially computed inside your component. Angular signals will give you extra simple reactivity in your template. With help of computed signals you can transform your input data without contract change, again in a reactive way!
  - Think about content projection as it underestimated feature. You can allow your users to choose what to project inside your component what makes it more flexible to changes
  - Your public API should be extra clear for your users, don't forget about types and interfaces as I said before. Stick to naming conventions for inputs and outputs. Better to rely on what Angular and popular libraries already providing.
  - Please don't abuse state it will make you more trouble in controlling your component. Sure you can use some state providers if needed, but then try to separate it from the other part of your app. Don't need to mix everything in one place. Same goes for localStorage or any other storage providers - use separate keys, so no other components will conflict with yours.
  - Don't forget about optimizations if your component can be used with a large data set. For example for table think about virtual scrolling(can be configurable via input). Also please consider memoization techiques if you have expensive operations. For some simple cases computed signals will do this job for you.
  - Change detection strategy 100% should be OnPush. No comments here.
  - Internationalization (i18n). Think what type of real users and from where will see your component. Date/number formatting, rtl support, text translation.
  - Think about styling and how it would be configurable. I suggest you to take a look at CSS custom properties as it one of the best soluions to let your users(devs) change something in the appearance.
  
