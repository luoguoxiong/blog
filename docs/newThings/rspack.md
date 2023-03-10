# Rspack

gitHub地址：https://github.com/web-infra-dev/rspack

文档地址：https://www.rspack.dev/zh/guide/introduction.html

Rspack是一个基于 Rust 的高性能构建引擎， 具备与 Webpack 生态系统的互操作性，可以被 Webpack 项目低成本集成，并提供更好的构建性能。

## 目标

* 快速的 Dev 启动性能。 npm run dev 是开发者每天需要运行很多次的命令，但大型项目每次都需要等待 10 分钟，这对于工程师来说非常痛苦，因此优化开发模式下启动的时间至关重要。

* 高效的 Build 性能。npm run build 经常在 CI/CD 环境中运行，它决定了应用生产交付的效率。有些应用在生产环境中需要 20 到 30 分钟的构建时间，如果能缩短这段时间，对开发流程也将非常有帮助。

* 灵活的配置。用户工程的配置非常灵活，不够统一。在之前的尝试中，将 Webpack 配置迁移到其他构建工具时，我们遇到了许多问题，因为其他构建工具的配置不如 Webpack 灵活。

* 生产环境的优化能力。在启用 Rspack 之前，我们尝试了社区内的各种方案，但它们都面临着一定程度的生产环境负优化，例如拆分包不够精细等。因此，优化生产环境的产物是我们不可放弃的功能。

## 当前状态

到今天（2023 年 3 月）为止 Rspack 已经开发了 11 个月，虽然 Rspack 仍处于比较早期的状态，且缺失了一些 webpack 的功能，但根据二八原则，目前的功能已经能够满足大多数项目的需求。同时，我们已经在内部的多个业务上完成了落地，取得了 5~10 编译性能的提升。目前的性能仍然存在较大提升空间，我们会持续对 Rspack 进行更深入的性能优化。

Rspack 已经完成了对 webpack 主要配置的兼容，并且适配了 webpack 的 loader 架构。目前，你已经可以在 Rspack 中无缝使用你熟悉的各种 loader，如 babel-loader、less-loader、sass-loader 等等。我们的长期目标是完整地支持 loader 特性，未来你可以在 Rspack 中使用那些更加复杂的 loader，如 vue-loader。 目前 Rspack 对缓存支持还比较简单，仅支持了内存级别的缓存，未来我们会建设更强的缓存能力，包括可迁移的持久化缓存，这将带来更大的想象空间，如在 monorepo 里不同的机器上都可以复用 Rspack 的云端缓存，提升大型项目的缓存命中率。

## 未来

虽然 Rspack 目前提供的能力已经能够满足大多数项目的使用，但是相比于 webpack 提供的完整能力，Rspack 仍然存在很大差距。因此，我们会根据社区的反馈，持续丰富 Rspack 的能力，从而满足更多构建场景的需求。

## 与 webpack 的区别
Webpack 是目前最为成熟的构建工具，有着活跃的生态，灵活的配置和丰富的功能，但是其最为人诟病的是其性能问题，尤其在一些大型项目上，构建花费的时间可能会达到几分钟甚至几十分钟，性能问题是目前 webpack 最大的短板。因此 Rspack 解决的问题核心就是 Webpack 的性能问题。 Rspack 比 Webpack 快得益于如下几方面：

* Rust 语言优势: Rspack 使用 Rust 语言编写， 得益于 Rust 的高性能编译器支持， Rust 编译生成的 Native Code 通常比 JavaScript 性能更为高效。

* 高度并行的架构: Webpack 受限于 JavaScript 对多线程的羸弱支持，导致其很难进行高度的并行化计算，而得益于 Rust 语言的并行化的良好支持， Rspack 采用了高度并行化的架构，如模块图生成，代码生成等阶段，都是采用多线程并行执行，这使得其编译性能随着 CPU 核心数的增长而增长，充分挖掘 CPU 的多核优势。

* 内置大部分的功能: 事实上 Webpack 本身的性能足够高效，但是因为 webpack 本身内置了较少的功能，这使得我们在使用 Webpack 做现代 Web App 开发时，通常需要配合很多的 plugin 和 loader 进行使用，而这些 loader 和 plugin 往往是性能的瓶颈，而 Rspack 虽然支持 loader 和 plugin，但是保证绝大部分常用功能都内置在 Rspack 内，从而减小 JS plugin | loader 导致的低性能和通信开销问题。

* 增量编译: 尽管 Rspack 的全量编译足够高效，但是当项目庞大时，全量的编译仍然难以满足 HMR 的性能要求，因此在 HMR 阶段，我们采用的是更为高效的增量编译策略，从而保证，无论你的项目多大，其 HMR 的开销总是控制在合理范围内。

## 与Vite区别
Vite 提供了很好的开发者体验，但 Vite 在生产构建中依赖了 Rollup ，因此与其他基于 JavaScript 的工具链一样，面临着生产环境的构建性能问题。

另外，Rollup 相较于 webpack 做了一些平衡取舍，在这里同样适用。比如，Rollup 缺失了 webpack 对于拆包的灵活性，即缺失了 optimization.splitChunks 提供的很多功能。

## 与 esbuild区别

我们在内部进行过大规模地将 esbuild 作为通用的 Web App 构建工具的实践，但是遇到如下一些问题：

* 缺乏对 JavaScript HMR 和增量编译的良好支持，这导致大型项目中的 HMR 性能较差。
* 拆包策略也非常原始，难以满足业务复杂多变的拆包优化需求。

## 与 Turbopack 的区别
Rspack 和 turbopack 都是基于 Rust 实现的 bundler，且都发挥了 Rust 语言的优势。

与 turbopack 不同的是，Rspack 选择了对 Webpack 生态兼容的路线，一方面，这些兼容可能会带来一定的性能开销，但在实际的业务落地中，我们发现对于大部分的应用来说，这些性能开销是可以接受的，另一方面，这些兼容也使得 Rspack 可以更好地与上层的框架和生态进行集成，能够实现业务的渐进式迁移。