# Referentially transparent GUIs

Back in late 2015 I started working on a project at a client to implement the
frontend for a new internal system. To tackle the problem we developed an
approach to UI programming that I wish would be more widely understood. Why?
Back then I wasn't a veteran frontend developer and, honestly, had doubts about
not seeing some obvious faults in our approach, which is why I was cautious
about promoting it. Myself and others have since then used it on several other
projects and spent a lot of time refining the approach in various ways. Working
as a consultant I have now also seen many different UI implementation approaches
on many different platforms and my appreciation for what we came up back then
has increased. Not a single technical issue that would have invalidated our
approach has turned up during the last 4.5 years and we have found gradual
improvements to various aspects of the approach.

Pretty much everything else I have seen since then has suffered from one or more
of typical problems:

<dl>

<dt>Not reflecting the view hierarchy</dt>

<dd>In the days of <a href="https://reactjs.org/">React</a> this may seem odd,
but in many UI projects there are places in the UI code where views are
constructed programmatically in such a way that one cannot simply take a glance
at the code and see the structure of the resulting view hierarchy. We can and
should do better: <blockquote>it is possible to make the structure of the
program match the structure of the problem being solved. &mdash; <a
href="http://jmct.cc/burge/1.html">Burge</a></blockquote></dd>

<dt>Not reflecting the application structure</dt>

<dd>In a related way, in many UI projects code has been organized into layers
rather than features. A single feature of the program is then spread over
several source files in different directories and it is often difficult to
understand what constitutes a specific feature.</dd>

<dt>Lost in plumbing</dt>

<dd>UI code tends to be full of intricate plumbing. Something happens or
changes, and the <a
href="https://awelonblue.wordpress.com/2012/07/01/why-not-events/">event needs
to be propagated through the UI</a>. More often than not, the amount of
hand-written propagation code is non-trivial and a source of major
difficulties. Almost every UI I have seen has leaked resources when you just
cycle through all the views or occasionally views become inconsistent with the
underlying state, because some update is not propagated properly.</dd>

<dt>Monolithic</dt>

<dd>Certain parts of UI code tend to be written in non-compositional ways such
that they accumulate details from logically separable components. This tends to
happen in pretty much every approach. In <a
href="https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93viewmodel">MVVM</a>
implementations the view models tend to become very complex. In <a
href="https://guide.elm-lang.org/architecture/">Elm</a> and <a
href="https://redux.js.org/introduction/core-concepts">Redux</a> the update
functions or reducers tend to accumulate <a
href="https://github.com/elmish/sample-react-todomvc/blob/6649454bda70b5ddf8acbe2f1f8d3128e8d18f5f/src/app.fs#L54-L118">more
and more repetitive branches</a>.  The major downside of this is that only views
or rendering code composes &mdash; everything else is tediously
hand-written.</dd>

</dl>

On the other, the approach we cooked up, has turned out to be:

<dl>

<dt>Modular</dt>

<dd>Components, including views and view models, can be <a
href="https://awelonblue.wordpress.com/2012/10/21/local-state-is-poison/">stateless</a>
and expressed as <a
href="https://en.wikipedia.org/wiki/Referential_transparency">referentially
transparent</a> stand-alone functions that can be used, tested, and &mdash; most
importantly &mdash; understood in isolation.</dd>

<dt>Plug-and-Play</dt>

<dd>Components, including views and view models, can generally be easily
"plugged in" to their surrounding context with minimal plumbing.  This is one of
the major insights of the approach and very much an explicit feature rather than
a lucky accident.</dd>

<dt>Refactorable</dt>

<dd>A complete GUI application can be written essentially as a single expression
or parts of it can be extracted into separate functions (view functions,
operations on view models, ...) without repercussions, because essentially
everything is expressed as <a
href="https://awelonblue.wordpress.com/2012/01/12/defining-declarative/">declarative</a>
referentially transparent expressions.  Simple (and still useful) components can
be as tiny as one line functions.  Complex components can be broken up into
understandable pieces that can be organized as desired.</dd>

<dt>Incremental</dt>

<dd>When the application state changes, updates to the UI are done efficiently
in <a href="https://en.wikipedia.org/wiki/Incremental_computing">an incremental
fashion</a> such that all and only the affected parts of the UI components are
updated to consistently reflect the state.  This is also an intentional feature
of the approach.</dd>

<dt>Portable</dt>

<dd>Our <a href="https://github.com/calmm-js">initial implementation</a> was
built on top of <a href="https://reactjs.org/">React</a>.  Later we have found
out that the same model can be easily built on top of pretty much any legacy UI
framework such as <a
href="https://en.wikipedia.org/wiki/Windows_Presentation_Foundation">WPF</a>,
plain <a
href="https://developer.mozilla.org/en-US/docs/Web/API/Document_Object_Model/Introduction">DOM</a>,
<a href="https://en.wikipedia.org/wiki/Cocoa_Touch">UIKit</a>, or Android
framework UI allowing one to easily use 100% of what those platforms offer out
of the box.</dd>

<dt>Interoperable</dt>

<dd>Components using the approach can generally be used as children or as
parents of "native" components.</dd>

</dl>

_Phew!_ I hope I didn't yet manage to put you into sleep! 😅

---

This story is continued in

- [Counters are not toys!](2020-05-11-counters-are-not-toys.md)
