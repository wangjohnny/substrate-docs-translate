---
title: Build the Substrate Kitties Chain
slug: /tutorials/v3/kitties/pt1
version: '3.0'
sideNav: kittiesWorkshop
section: tutorials
category: FRAME
keywords:
  - nft
  - kitties
difficulty: 2
duration: 2 Hours
relevantSkills:
  - FRAME
  - Rust
---
img-QnbLcVHm-1651399324385)(../../../../src/images/tutorials/11-kitties-workshop/kitties-tutorial.png)]

Welcome to the Substrate Kitties workshop.
This workshop teaches you everything you need to know about building a blockchain that handles the creation and ownership of non-fungible tokens (NFTs) called Substrate Kitties.The workshop is split into two parts:
欢迎来到Substrate Kitties课程。
本课程将向您介绍如何构建一个可以创建并持有非同质化代币（NFT，这个NFT名称为Substrate Kitties）的区块链。课程分为部2部分：
Part I describes how to build the Kitties pallet, including the functionality to interact with the Kitties you create.
第一部分描述如何构建Kitties pallet，这个pallet功能是如何与你所创建的Kitties交互。
[Part II](/tutorials/v3/kitties/pt2) describes how to develop a front-end to interact with the Substrate Kitties blockchain from Part I.
[Part II](/tutorials/v3/kitties/pt2) 描述如何开发一个前端，这个前端需要与Part I 的Substrate Kitties区块链进行交互。
## Tutorial objectives -- 教程目标
- Learn basic patterns for building and running a Substrate node. 学习构建并运行一个Substrate节点的基本模式。
- Write and integrate a custom FRAME pallet to your runtime. 写一个自定义FRAME pallet，并与你的节点runtime集成好。
- Learn how to create and update storage items. 学习如何创建与更新存储xixianxiang项(items)。
- Write pallet extrinsics and helper functions. 编写pallet交易（extrinsics）与辅助函数。
- Use the PolkadotJS API to connect a Substrate node to a custom a front-end. 自定义的前端程序用Polkadot API连接到Substrate节点。

This tutorial assumes that you have already installed the prerequisites for building with Substrate on your machine.
本教程假设您已经在您的机器上安装了使用 Substrate 构建的先决条件。
Make sure you have configured your environment for Substrate development by installing [Rust and the Rust toolchain](/v3/getting-started/installation).
通过安装 [Rust and the Rust toolchain](/v3/getting-started/installation) 确保您已经为 Substrate 开发配置好了环境

### What we're building -- 我们所要构建的

We'll intentionally keep things simple so that you can decide on how you'd like to improve your Substrate Kitties chain later on.
For the purposes of what we're building, Kitties really can only do the following things:
我们尽量让内容简单一些，方便您以后可以决定如何改进您的 Substrate Kitties 链。
为了达到我们的构建目的，Kitties应用只做以下事情：

- Be created either by some original source or by being bred using existing Kitties. 通过一些初始资源或者使用已有猫咪来繁殖猫咪。

- Be sold at a price set by their owner. 所有者可以设定一个价格来出售猫咪。

- Be transferred from one owner to another. 所有者可以把猫咪转让给别人。

### What we won't cover -- 我们不会涉及的
The following items fall outside the scope of this tutorial:
以下内容不在本教程的范围内：
- Writing tests for our pallet.
- 为pallet编写测试用例。
- Using correct weight values.
- 使用正确的权重（weight）数值。

You can refer to the [how-to guides](/how-to-guides/v3) on how to integrate these once you've completed this workshop.
完成本课程后，您可以参考 [how-to guides](/how-to-guides/v3) 来了解如何在应用中集成以上内容。

Follow each step at your own pace &mdash; the goal is for you to learn and the best way to do that is to try it yourself!
定好自己的学习目标，按照自己的节奏，参考教程的每个步骤 ，最好的方法就是自己不停地试验！

Before moving on from one section to the next, make sure your pallet builds without any error.
Use the [template files](https://github.com/substrate-developer-hub/substrate-docs/tree/main/static/assets/tutorials/kitties-tutorial) to help you complete each part.
在从一个步骤移动到下一个步骤前，请确保您的pallet构建没有任何错误。
使用[模板文件](https://github.com/substrate-developer-hub/substrate-docs/tree/main/static/assets/tutorials/kitties-tutorial)辅助你完成每一个部分。

If you are stuck you can refer to the complete source code on the [Substrate node template repository \`tutorials/solutions/kitties\` branch](https://github.com/substrate-developer-hub/substrate-node-template/tree/tutorials/solutions/kitties).
Most of the code changes will be in `/pallets/kitties/src/lib.rs`.
如果被问题卡住了，可以参考 [Substrate note template仓库的\`tutorials/solutions/kitties\`分支](https://github.com/substrate-developer-hub/substrate-node-template/tree/tutorials/solutions/kitties)上的完整源代码。
大多数代码修改都是在 `/pallets/kitties/src/lib.rs` 文件中。

## Basic set-up -- 基本设置

Before we can start making Kitties, we first need to do a little groundwork.
This part covers the basic patterns involved with using the Substrate node template to set up a custom pallet and include a simple storage item.
在我们开始构建Kitties应用前，我们首先需要做一些基础工作。
这部分介绍了一些使用 Substrate node template 的基本模式，这些模式涉及自定义pallet的设置与如何包含一个简单的存储项。

### Set-up your template node -- 设定你的 template node

The [Substrate node template][substrate-node-template] provides us with a customizable blockchain node, with built-in networking and consensus layers included.
All we need to focus on is building out the logic of our [runtime][runtime-kb] and [pallets][pallets-kb].
[Substrate node template][substrate-node-template] 为我们提供了一个可定制的区块链节点，包括内置的网络层和共识层。
我们需要关注的是构建我们的 [runtime][runtime-kb] 和 [pallets][pallets-kb] 的逻辑。


To kick things off, we need to set-up our project name and dependencies.
We'll use a CLI tool called [kickstart][kickstart-tool] to easily rename our node template.
首先，我们需要设置我们的项目名称和依赖项。
我们将使用一个名为 [kickstart][kickstart-tool] 的 命令行（CLI） 工具来重命名我们的node template。

1. Install it by running `cargo install kickstart`. 
	通过运行 `cargo install kickstart` 来安装。

1. Once `kickstart` is installed, in the root directory of your local workspace run the following command: 
	安装好 `kickstart` 后，在工作空间的根目录下运行下列命令：

   ```bash
   kickstart https://github.com/sacha-l/kickstart-substrate
   ```

   This command will clone a copy of the most recent node template and ask how you would like to call your node and pallet.
   这个命令将克隆出最新的node template代码，并询问您希望如何调用node与pallet。

1. Type in:
	输入：

   - `kitties` - as the name of our node. The node will be named as `node-kitties`. 节点（node）名称， 这个节点将被命名为“node-kitties”。
   - `kitties` - as the name of your pallet. The pallet will be named as `pallet-kitties`. `kitties` - pallet名称。 这个pallet将被命名为“pallet-kitties”。

   This will create a directory called `kitties` with a copy of the [Substrate node template][substrate-node-template] containing the name changes that correspond our template node, runtime, and pallet.
   这将创建一个名为 `kitties` 的目录，目录中包含 [Substrate node template][substrate-node-template] 的副本，副本中包含了与template node、runtime与pallet对应的名称修改。

1. Open the `kitties` directory in your favorite code editor and rename it as `kitties-tutorial` &mdash; or whatever name you like to help keep your work organized. 
	用您喜欢的代码编辑器，打开`kitties`目录并将其重命名为`kitties-tutorial`（或任何您喜欢的名称），以帮助您保持工作井井有条。

   Notice the directories that the `kickstart` command modified:
   注意 `kickstart` 命令修改的目录：

   - **/node/** - This folder contains all the logic that allows your node to interact with your runtime and RPC clients.
	**/node/** - 此文件夹包含node与runtime、node与RPC 客户端的所有交互逻辑。
   - **/pallets/** - Here's where all your custom pallets live.
	**/pallets/** - 这里是您所有自定义pallet的所在地。
   - **/runtime/** - This folder is where all pallets (both custom "internal" and "external" ones) are aggregated and implemented for the chain's runtime.
	**/runtime/** - 此文件夹是所有pallet（包含自定义的“内部”和“外部”）的聚合之地，并实现为区块链的runtime。

1. In `runtime/src/lib.rs`, you'll also notice that the instance of our modified template pallet name remains `TemplateModule`.
	在 `runtime/src/lib.rs` 中，您还会注意到我们修改后的模板pallet名称的实例仍然包含 `TemplateModule`。
 
   Change it to `SubstrateKitties`:
   修改为 `SubstrateKitties`:

   ```rust
   construct_runtime!(
   	// --snip
   	{
   	    // --snip
   	    SubstrateKitties: pallet_kitties,
   	}
   );
   ```

### Write the `pallet_kitties` scaffold -- 编写 `pallet_kitties` 脚手架代码

Let's take a glance at the folder structure of our workspace:
我们一起来看一下工作空间的文件结构：

```bash
kitties-tutorial           <--  The name of our project directory
|
+-- node
|
+-- pallets
|   |
|   +-- kitties
|       |
|       +-- Cargo.toml
|       |
|       +-- src
|           |
|           +-- benchmarking.rs     <-- Remove file
|           |
|           +-- lib.rs              <-- Remove contents
|           |
|           +-- mock.rs             <-- Remove file
|           |
|           +-- tests.rs            <-- Remove file
|
+-- Cargo.toml
```

> You can go ahead and remove `benchmarking.rs`, `mock.rs` and `tests.rs`.
	您可以先删除 `benchmarking.rs`、`mock.rs` 和 `tests.rs`。
> **We won't be learning about using these in this tutorial.**
	**我们不会在本教程中学习如何使用这些。**
> Have a look at [this how-to guide](/how-to-guides/v3/testing/transfer-function) if you're curious to learn how testing works.
	如果您想了解测试的工作原理，请查看 [this how-to guide](/how-to-guides/v3/testing/transfer-function)。

[Pallets][pallets-kb] in Substrate are used to define runtime logic.
In our case, we'll be creating a single pallet that manages all of the logic of our Substrate Kitties application.
Substrate 中的 [Pallets][pallets-kb] 用于定义运行时（runtime）逻辑。
在我们的例子中，我们将创建一个独立的pallet来管理我们的 Substrate Kitties 应用程序的所有逻辑。

Notice that our pallet's directory `/pallets/kitties/` is not the same as our pallet's name.
The name of our pallet as Cargo understands it is `pallet-kitties`.
请注意，我们的pallet目录 `/pallets/kitties/` 是与我们的pallet名称不同的。
按照Cargo的理念，我们的pallet名称是“pallet-kitties”。

Let's lay out the basic structure of our pallet by outlining the parts inside `pallets/kitties/src/lib.rs`.
通过 `pallets/kitties/src/lib.rs`文件中的内容，我们可以勾勒出我们自己pallet的基础结构。

Every FRAME pallet has:
每个FRAME pallet包含：

- A set of `frame_support` and `frame_system` dependencies.
	`frame_support` 和 `frame_system` 依赖项。
- Required [attribute macros][macros-kb] (i.e. configuration traits, storage items and function calls).
	必需的 [attribute macros][macros-kb]（即配置trait、存储项和函数调用）。

<Message
  type={`yellow`}
  title={`Information`}
  text={`
We'll be updating additional dependencies as we progress through the next parts of this tutorial.
随着本教程进行到下一部分时，我们将更新其他依赖项。
  `}
/>

Here's the most bare-bones version of the Kitties pallet we will be building in this tutorial.
It contains the starting point for adding code for the next sections of this tutorial.
Just like the [helper files](https://github.com/substrate-developer-hub/substrate-docs/tree/main/static/assets/tutorials/kitties-tutorial) for this tutorial, it contains comments marked with **TODO** to indicate code we will be writing later, and **ACTION** to indicate code that will be written in the current section.
这将是我们在本教程中构建的最简单 pallet 版本。
它包含了为下一部分添加代码的起点。
就像本教程的 [辅助文件](https://github.com/substrate-developer-hub/substrate-docs/tree/main/static/assets/tutorials/kitties-tutorial)，它包含了标有**TODO**的注释（ 表示我们稍后将编写代码）与**ACTION**注释（表示将在当前编写代码）。

1. Paste the following code in `/pallets/kitties/src/lib.rs`:
    粘贴以下代码到文件`/pallets/kitties/src/lib.rs`：

   ```rust
   #![cfg_attr(not(feature = "std"), no_std)]

   pub use pallet::*;

   #[frame_support::pallet]
   pub mod pallet {
   	use frame_support::{
   		sp_runtime::traits::{Hash, Zero},
   		dispatch::{DispatchResultWithPostInfo, DispatchResult},
   		traits::{Currency, ExistenceRequirement, Randomness},
   		pallet_prelude::*
   	};
   	use frame_system::pallet_prelude::*;
   	use sp_io::hashing::blake2_128;

   	// TODO Part II: Struct for holding Kitty information.

   	// TODO Part II: Enum and implementation to handle Gender type in Kitty struct.

   	#[pallet::pallet]
   	#[pallet::generate_store(pub(super) trait Store)]
   	pub struct Pallet<T>(_);

   	/// Configure the pallet by specifying the parameters and types it depends on.
   	#[pallet::config]
   	pub trait Config: frame_system::Config {
   		/// Because this pallet emits events, it depends on the runtime's definition of an event.
   		type Event: From<Event<Self>> + IsType<<Self as frame_system::Config>::Event>;

   		/// The Currency handler for the Kitties pallet.
   		type Currency: Currency<Self::AccountId>;

   		// TODO Part II: Specify the custom types for our runtime.

   	}

   	// Errors.
   	#[pallet::error]
   	pub enum Error<T> {
   		// TODO Part III
   	}

   	#[pallet::event]
   	#[pallet::generate_deposit(pub(super) fn deposit_event)]
   	pub enum Event<T: Config> {
   		// TODO Part III
   	}

   	// ACTION: Storage item to keep a count of all existing Kitties.

   	// TODO Part II: Remaining storage items.

   	// TODO Part III: Our pallet's genesis configuration.

   	#[pallet::call]
   	impl<T: Config> Pallet<T> {

   		// TODO Part III: create_kitty

   		// TODO Part III: set_price

   		// TODO Part III: transfer

   		// TODO Part III: buy_kitty

   		// TODO Part III: breed_kitty
   	}

   	// TODO Part II: helper function for Kitty struct

   	impl<T: Config> Pallet<T> {
   		// TODO Part III: helper functions for dispatchable functions

   		// TODO: increment_nonce, random_hash, mint, transfer_from

   	}
   }
   ```

2. Notice that we're using `sp_io` in our pallet.
   Ensure that this is declared as a dependency in your pallet's `Cargo.toml` file:
   注意pallet中正在使用`sp_io`文件。
   确保在你pallet项目的`Cargo.toml`文件中，声明了这个依赖项：

   ```toml
   sp-io = { default-features = false, git = "https://github.com/paritytech/substrate.git", branch = "polkadot-v0.9.19" }
   ```

3. Now try running the following command to build your pallet. We won't build the entire chain just yet because we haven't yet implemented the `Currency` type in our runtime.
   At least we can check that there are no errors in our pallet so far:
   现在尝试运行以下命令来构建您的pallet。 我们还不会构建整个区块链，因为我们还没有在我们的runtime中实现`Currency`类型。
    我们可以检查到目前为止我们的pallet中没有错误：

   ```bash
   cargo build -p pallet-kitties
   ```

You'll notice the Rust compiler giving you warnings about unused imports.
That's fine!
Just ignore them &mdash; we're going to be using those imports in the later parts of the tutorial.
你会注意到 Rust 编译器会给你关于未使用的导入警告。
没关系！
忽略它们吧—— 我们将在教程的后面部分使用这些导入。

### Add storage items -- 添加存储项

Let's add the most simple logic we can to our runtime: a function that stores a variable in our runtime.
To do this we'll use [`StorageValue`][storagevalue-rustdocs] from Substrate's [storage API][storage-api-rustdocs] which is a trait that depends on the [storage macro][storage-macro-kb].
让我们向runtime添加最简单的逻辑：在runtime中存储一个变量的函数。
为此，我们将使用一个叫做 [`StorageValue`][storagevalue-rustdocs] from Substrate's [storage API][storage-api-rustdocs] 的trait，这个trait依赖[storage macro][storage-macro-kb]。

All that means for our purposes is that for any storage item we want to declare, we must include the `#[pallet::storage]` macro beforehand.
Learn more about declaring storage items [here](/v3/runtime/storage#declaring-storage-items).
就我们的目的而言，这意味着对于我们要声明的任何存储项，我们必须事先包含`#[pallet::storage]`宏。
了解有关声明存储项的更多信息 [这里](/v3/runtime/storage#declaring-storage-items)。

In `pallets/kitties/src/lib.rs`, replace the ACTION line with:
用以下内容替换`pallets/kitties/src/lib.rs`文件中的ACTION行：

```rust
#[pallet::storage]
#[pallet::getter(fn count_for_kitties)]
/// Keeps track of the number of Kitties in existence.
pub(super) type CountForKitties<T: Config> = StorageValue<_, u64, ValueQuery>;
```

This creates a storage item for our pallet to keep track of the total count of Kitties in existence.
这为我们的pallet创建了一个存储项，以跟踪现有猫咪的总数。

### Add Currency implementation -- 添加货币实现

Before we proceed with building our node, we need to add the Currency type to our pallet's runtime implementation. In `runtime/src/lib.rs`, add the following:
在继续构建node之前，我们需要将 Currency 类型添加到pallet的runtime实现中。 在 `runtime/src/lib.rs` 中，添加以下内容：

```rust
impl pallet_kitties::Config for Runtime {
	type Event = Event;
	type Currency = Balances; // <-- Add this line
}
```

Now build your node and make sure you don't have any errors. This will take a little while at first.
现在构建您的node，并确保您没有遇到任何错误。 假如是第一次构建，这将需要一些时间的。

```bash
cargo build --release
```

🎉**_Congratulations!_**🎉

You've completed the first part of this series.
At this stage, you've learnt the various patterns for:
您已经完成了这个系列的第一部分。
在这个阶段，您学习了以下各种模式：

- Customizing the Substrate node template and including a custom pallet.
	自定义 Substrate node template，并包含一个自定义的pallet。
- Building a Substrate chain and checking that a target pallet compiles.
	构建 Substrate 链并检查目标pallet是否能通过编译。
- Declaring a single value \`u64\` storage item.
	声明单个值 \`u64\` 存储项。

## Uniqueness, custom types and storage maps -- 唯一性、自定义类型和存储映射

You're ready to dive into some pillar concepts for developing pallets with FRAME (**Framework for Runtime Aggregation of Modularized Entities**), including writing [a storage struct](/how-to-guides/v3/pallet-design/storage-value) and [implementing the randomness trait](/how-to-guides/v3/pallet-design/randomness).
您已经深入了解了使用 FRAME （**一个把各种模块化实体聚合为runtime的框架**）来开发pallet的一些关键概念，包括编写一个 [存储struct](/how-to-guides/v3/pallet-design/storage-value) and [实现随机数的trait](/how-to-guides/v3/pallet-design/randomness)。

Use the [helper code](https://github.com/substrate-developer-hub/substrate-docs/tree/main/static/assets/tutorials/kitties-tutorial/02-create-kitties.rs) provided below to help you complete each step.
This will be the basis of the next few steps.
使用下面提供的[辅助代码](https://github.com/substrate-developer-hub/substrate-docs/tree/main/static/assets/tutorials/kitties-tutorial/02-create-kitties.rs) 帮助您完成每一步。
这将是接下来几个步骤的基础。

Update your pallet code with the following code snip (skip this step if you prefer not to use the template code):
使用以下代码片段更新您的pallet代码（如果您不想使用模板代码，请跳过此步骤）：

```rust
#![cfg_attr(not(feature = "std"), no_std)]

pub use pallet::*;

#[frame_support::pallet]
pub mod pallet {
	use frame_support::pallet_prelude::*;
	use frame_system::pallet_prelude::*;
	use frame_support::{
		sp_runtime::traits::Hash,
		traits::{ Randomness, Currency, tokens::ExistenceRequirement },
		transactional
	};
	use sp_io::hashing::blake2_128;

	#[cfg(feature = "std")]
	use frame_support::serde::{Deserialize, Serialize};

	// ACTION #1: Write a Struct to hold Kitty information.

	// ACTION #2: Enum declaration for Gender.

	// ACTION #3: Implementation to handle Gender type in Kitty struct.

	#[pallet::pallet]
	#[pallet::generate_store(pub(super) trait Store)]
	pub struct Pallet<T>(_);

	/// Configure the pallet by specifying the parameters and types it depends on.
	#[pallet::config]
	pub trait Config: frame_system::Config {
		/// Because this pallet emits events, it depends on the runtime's definition of an event.
		type Event: From<Event<Self>> + IsType<<Self as frame_system::Config>::Event>;

		/// The Currency handler for the Kitties pallet.
		type Currency: Currency<Self::AccountId>;

		// ACTION #5: Specify the type for Randomness we want to specify for runtime.

		// ACTION #9: Add MaxKittyOwned constant
	}

	// Errors.
	#[pallet::error]
	pub enum Error<T> {
		// TODO Part III
	}

	// Events.
	#[pallet::event]
	#[pallet::generate_deposit(pub(super) fn deposit_event)]
	pub enum Event<T: Config> {
		// TODO Part III
	}

	#[pallet::storage]
	#[pallet::getter(fn count_for_kitties)]
	pub(super) type CountForKitties<T: Config> = StorageValue<_, u64, ValueQuery>;

	// ACTION #7: Remaining storage items.

	// TODO Part IV: Our pallet's genesis configuration.

	#[pallet::call]
	impl<T: Config> Pallet<T> {

		// TODO Part III: create_kitty

		// TODO Part IV: set_price

		// TODO Part IV: transfer

		// TODO Part IV: buy_kitty

		// TODO Part IV: breed_kitty
	}

	//** Our helper functions.**//

	impl<T: Config> Pallet<T> {

		// ACTION #4: helper function for Kitty struct

		// TODO Part III: helper functions for dispatchable functions

		// ACTION #6: function to randomly generate DNA

		// TODO Part III: mint

		// TODO Part IV: transfer_kitty_to
	}
}
```

Along with this code, we'll need to import `serde`.
Add this to your pallet's Cargo.toml file, using the matching version as Substrate upstream.
除了这段代码，我们还需要导入 `serde`。
将其添加到您pallet的 Cargo.toml 文件中，使用匹配的版本作为最新的 Substrate。

### Scaffold Kitty struct -- 编写Kitty结构的框架代码

A Struct in Rust is a useful construct to help store data that have things in common.
For our purposes, our Kitty will carry multiple properties which we can store in a single struct instead of using separate storage items.
This comes in handy when trying to optimize for storage reads and writes so our runtime can perform less read/writes to update multiple values.
Read more about storage best practices [here][storage-best-practice-kb].
Rust 中的 Struct 是一种有用的构造，有助于存储具有共性的数据。
出于我们的目的，我们的 Kitty 将携带多个属性，我们可以将它们存储在单个结构中，而不是每个属性使用单独的存储项。
这在尝试优化存储读取和写入时会派上用场，因此我们的runtime可以执行更少的读取/写入来更新多个值。
阅读更多关于存储最佳实践的信息 [这里][storage-best-practice-kb]。

#### What information to include -- 包含哪些信息

Let's first go over what information a single Kitty will carry:
我们先来看看单个 Kitty 会包括哪些信息：

- **`dna`**: the hash used to identify the DNA of a Kitty, which corresponds to its unique features.
  DNA is also used to breed new Kitties and to keep track of different Kitty generations.
	**`dna`**：用于识别 Kitty 的 DNA 的哈希值，与它的独特特征相对应。
   DNA 还用于繁殖新的猫咪并跟踪不同的猫咪世代。
- **`price`**: this is a balance that corresponds to the amount needed to buy a Kitty and
  set by its owner.
	**`price`**：这是购买 Kitty 所需的金额或者所有者设定的售出价格。
- **`gender`**: an enum that can be either `Male` or `Female`.
	**`gender`**：一个可以是 `Male` 或 `Female` 的枚举。
- **`owner`**: an account ID designating a single owner.
	**`owner`**：指定单个所有者的帐户 ID。

#### Sketching out the types held by our struct -- 预估我们的结构所持有的类型

Looking at the items of our struct from above, we can deduce the following types:
从上节内容中结构体中的条目，我们可以推断出以下类型：

- **`[u8; 16]`** for `dna` - to use 16 bytes to represent a Kitty's DNA.
	**`[u8; 16]`** for `dna` - 使用 16 个字节来表示 Kitty 的 DNA。
- **`BalanceOf`** for `price` - this is a custom type using FRAME's [`Currency` trait][currency-frame].
	**`BalanceOf`** for `price` - 这是使用 FRAME 中 [`Currency` trait][currency-frame] 的自定义类型。
- **`Gender`** for `gender` - we are going to create this!
	**`Gender`** for `gender` - 我们需要创建这个！

First, we will need to add in our custom types for `BalanceOf` and `AccountOf` before we declare our struct.
Replace ACTION #1 with the following snippet:
首先，在声明结构之前，我们需要为 `BalanceOf` 和 `AccountOf` 添加自定义类型。
用以下代码段替换ACTION #1：

```rust
type AccountOf<T> = <T as frame_system::Config>::AccountId;
type BalanceOf<T> =
	<<T as Config>::Currency as Currency<<T as frame_system::Config>::AccountId>>::Balance;

// Struct for holding Kitty information.
#[derive(Clone, Encode, Decode, PartialEq, RuntimeDebug, TypeInfo, MaxEncodedLen)]
#[scale_info(skip_type_params(T))]
#[codec(mel_bound())]
pub struct Kitty<T: Config> {
	pub dna: [u8; 16],
	pub price: Option<BalanceOf<T>>,
	pub gender: Gender,
	pub owner: AccountOf<T>,
}
```

<br />
<Message
  type="gray"
  title="Note"
  text={`
We define \`<BalanceOf<T>>\` and \`AccountOf<T>\` types, and use them in the \`Kitty\`. If you
wonder what the first line means in Rust, it is to define a type alias \`AccountOf<T>\` which is
just a shorthand pointing to the associated type \`AccountId\` of trait \`frame_system::Config\`
that generic type \`T\` is required to be bound of.\n\n
More about this type of syntax is covered in [the Rust book](https://doc.rust-lang.org/book/ch19-03-advanced-traits.html#specifying-placeholder-types-in-trait-definitions-with-associated-types). 我们定义了\`<BalanceOf<T>>\` 和 \`AccountOf<T>\` 类型，并在 \`Kitty\` 中使用它们。 如果你想知道 Rust 中的第一行是什么意思，它是定义一个类型别名 \`AccountOf<T>\`，它是指向 trait \`frame_system::Config\` 的关联类型 \`AccountId\` 的速写，这个类型需要绑定泛型\`T\`。\n\n更多关于这种类型的语法在 [the Rust book](https://doc.rust-lang.org/book/ch19-03-advanced-traits.html#specifying-placeholder-types-in-trait-definitions-with-associated-types)。
`}
/>

Notice how we use the `derive` macro to include [various helper traits][prelude-traits-rustdocs] for using our struct.
We'll need to add `TypeInfo` in order to give our struct access to this trait.
Add the following line at the top of your pallet:
请注意，为了使用我们的结构（strutct），我们如何使用 `derive` 宏来包含 [各种辅助trait][prelude-traits-rustdocs] 。
我们需要添加 `TypeInfo` 以使我们的结构可以访问此 trait。
在你的pallet的顶部添加以下内容：

```rust
use scale_info::TypeInfo;
```

For type `Gender`, we will need to build out our own custom enum and helper functions.
对于 `Gender`类型，我们需要定义自己的枚举类型和辅助函数。

### Write a custom type for `Gender` -- 为 `Gender` 编写自定义类型

We have just created a struct that requires a custom type called `Gender`.
This type will handle an enum defining our Kitty's gender.
To create it, you'll build out the following parts:
我们刚刚创建了一个结构，它需要一个名为 `Gender` 的自定义类型。
这种类型将是我们定义猫咪性别的枚举类型。
要创建它，您将构建以下部分：

- **An enum declaration**, which specifies `Male` and `Female` values.
	**枚举声明**，指定 `Male` 和 `Female` 值。
- **Implement a helper function** for our Kitty struct.
	**为我们的 Kitty 结构实现一个辅助函数**。

1. #### Declare the custom enum -- 声明自定义枚举

   Replace ACTION item #2 with the following enum declaration:
   用下列的枚举声明替换ACTION #2:

   ```rust
   #[derive(Clone, Encode, Decode, PartialEq, RuntimeDebug, TypeInfo, MaxEncodedLen)]
   #[cfg_attr(feature = "std", derive(Serialize, Deserialize))]
   pub enum Gender {
   	Male,
   	Female,
   }
   ```

   Notice the use of the [derive macro][derive-macro-rust] which must precede the enum declaration.
   This wraps our enum in the data structures it will need to interface with other types in our runtime.
   In order to use `Serialize` and `Deserialize`, you will need to add the `serde` crate in `pallets/kitties/Cargo.toml`, using the matching version as Substrate upstream.
	注意在枚举声明，必须使用 [derive macro][derive-macro-rust]。
    这将我们的枚举包装在数据结构中，它需要在我们的runtime中与其他类型进行交互。
    为了使用 `Serialize` 和 `Deserialize`，您需要在 `pallets/kitties/Cargo.toml` 中添加 `serde` crate，使用匹配的版本作为Substrate upstream。

   Great, we now know how to create a custom struct.
   But what about providing a way for a Kitty struct to be _assigned_ a gender value?
   For that we need to learn one more thing.
	太好了，我们现在知道如何创建自定义结构了。
    但是如何提供一种方法给 Kitty 结构设置一个性别值呢？
    为此，我们需要再学习一件事。

#### Implement a helper function for our Kitty struct--为我们的Kitty结构实现是个辅助函数

   Configuring a struct is useful in order to pre-define a value in our struct.
   For example, when setting a value in relation to what another function returns.
   In our case we have a similar situation where we need to configure our Kitty struct in such a way that sets `Gender` according to a Kitty's DNA.
	为了在我们的结构中预先定义一个值，配置一个结构是很有用的。
    例如，在设置与另一个函数返回的值相关的值时。
    在我们的例子中，我们有类似的情况，我们需要配置我们的 Kitty 结构，以便根据 Kitty 的 DNA 设置`Gender`。

   We'll only be using this function when we get to [creating Kitties](#write-the-create_kitty-dispatchable).
   Regardless, let us learn how to write it now and get it out of the way.
   We'll create a public function called `gen_gender` that returns the `Gender` type and uses a random function to choose between `Gender` enum values.
	我们只会在[创建 Kitty](#write-the-create_kitty-dispatchable) 时使用这个函数。
    无论如何，让我们现在就学习如何编写，并完成这个函数。
    我们将创建一个名为 `gen_gender` 的公共函数，它返回 `Gender` 类型并使用随机函数在 `Gender` 枚举值之间进行选择。

   Replace ACTION #4 with the following code snippet:
   用下列代码片段替换ACTION #4：

   ```rust
   fn gen_gender() -> Gender {
   	let random = T::KittyRandomness::random(&b"gender"[..]).0;
   	match random.as_ref()[0] % 2 {
   		0 => Gender::Male,
   		_ => Gender::Female,
   	}
   }
   ```

   Now whenever `gen_gender()` is called inside our pallet, it will return a pseudo random enum value for `Gender`.
   现在，每当在我们的pallet中调用 `gen_gender()` 时，它都会返回 `Gender` 的伪随机枚举值。

### Implement on-chain randomness -- 实现链上随机性

If we want to be able to tell these Kitties apart, we need to start giving them unique properties!
In the previous step, we've made use of `KittyRandomness` which we haven't actually defined yet.
Let's get to it.
如果我们希望能够区分这些猫咪，我们需要开始赋予它们某些独特的属性！
在上一步中，我们使用了尚未实际定义的“KittyRandomness”。
让我们开始吧。

We'll be using the [Randomness trait][randomness-rustdocs] from `frame_support` to do this.
It will be able to generate a random seed which we'll create unique Kitties with as well as breed new ones.
我们将使用 `frame_support` 中的 [Randomness trait][randomness-rustdocs] 来执行此操作。
它将能够生成一个随机种子，我们将用这个种子来创建独特的猫咪，并繁殖新的猫咪。

1. In your pallet's configuration trait, define a new type bound by the `Randomness` trait.
	在您的pallet的配置trait中，定义一个受 `Randomness` trait 约束的新类型。

   The `Randomness` trait from `frame_support` requires specifying it with a parameter to replace the `Output` and `BlockNumber` generics.
   Take a look at [the documentation][randomness-rustdocs] and the source code implementation to understand how this works.
   For our purposes, we want the output of functions using this trait to be [`Blake2 128-bit hash`][blake2-rustdocs] which you'll notice should already be declared at the top of your working codebase.
	需要用参数指定来自`frame_support` 中的 `Randomness` trait，以替换 `Output` 和 `BlockNumber` 泛型。
    查看 [文档][randomness-rustdocs] 和源代码实现以了解其工作原理。
    为了达到目的，我们希望使用trait函数的输出是 [`Blake2 128-bit hash`][blake2-rustdocs] ，你会注意到你应该已经在现存代码的头部声明了这些内容。
    
   Replace the ACTION #5 line with:
   用下列代码替换ACTION #5行：
   ```rust
   type KittyRandomness: Randomness<Self::Hash, Self::BlockNumber>;
   ```

1. Specify the actual type in your runtime.
	指定runtime里的实际类型。
	
   Given that we have added a new type in the configuration of our pallet, we need to config our runtime to set its concrete type.
   This could come in handy if ever we want to change the algorithm that `KittyRandomness` is using, without needing to modify where it's used inside our pallet.
	由于我们在pallet配置中添加了一个新类型，所以我们需要配置runtime，来设置这个新类型的具体类型。
    如果我们想更改`KittyRandomness` 正在使用的算法，而不需要在pallet里修改它的使用位置，这可能会派上用场。

   To showcase this point, we're going to set the `KittyRandomness` type to an instance of [FRAME's `RandomnessCollectiveFlip`][randomness-collective-flip-frame].
   Conveniently, the node template already has an instance of the `RandomnessCollectiveFlip` pallet.
   All you need to do is **set the `KittyRandomness` type in your runtime, inside `runtime/src/lib.rs`**:
   为了展示这一点，我们将 `KittyRandomness` 类型设置为 [FRAME 的 `RandomnessCollectiveFlip`][randomness-collective-flip-frame] 的一个实例。
   方便的是，node template已经有一个 `RandomnessCollectiveFlip` pallet的实例。
   您需要做的就是 **在运行时中的`runtime/src/lib.rs`中设置`KittyRandomness`类型**：

   ```rust
   impl pallet_kitties::Config for Runtime {
   	type Event = Event;
   	type Currency = Balances;
   	type KittyRandomness = RandomnessCollectiveFlip; // <-- ACTION: add this line.
   }
   ```

   Here we have abstracted out the randomness generation implementation (`RandomnessCollectiveFlip`) from its interface (`Randomness<Self::Hash, Self::BlockNumber>` trait).
   Check out this [how-to guide](/how-to-guides/v3/pallet-design/randomness) on implementing randomness in case you get stuck.
   在这里，我们从其接口（`Randomness<Self::Hash, Self::BlockNumber>` trait）中抽象出生成随机数的实现（`RandomnessCollectiveFlip`）。
   看看这个 [how-to-guide](/how-to-guides/v3/pallet-design/randomness) 了解如何实现随机性，以防您遇到困难。
   
1. Generate random DNA
	生成随机DNA

   Generating DNA is similar to using randomness to randomly assign a gender type.
   The difference is that we'll be making use of `blake2_128` we imported in the previous part.
   We will also be using the [extrinsic_index](https://docs.rs/frame-system/latest/frame_system/pallet/struct.Pallet.html#method.extrinsic_index) from the `frame_system` pallet in order to generate different hashes if we call this function more then once within the same block.
   生成 DNA 类似于使用随机性来随机分配性别类型。
   不同之处在于我们将使用在前一部分中导入的 `blake2_128`。
   我们还将使用 `frame_system` pallet中的 [extrinsic_index](https://docs.rs/frame-system/latest/frame_system/pallet/struct.Pallet.html#method.extrinsic_index) ，为了生成不同的哈希值，我们可以在同一个块中多次调用这个函数。
   Replace the ACTION #6 line with:
   用下面代码替换ACTION #6行：

   ```rust
   fn gen_dna() -> [u8; 16] {
   	let payload = (
   		T::KittyRandomness::random(&b"dna"[..]).0,
   		<frame_system::Pallet<T>>::extrinsic_index().unwrap_or_default(),
   		<frame_system::Pallet<T>>::block_number(),
   	);
   	payload.using_encoded(blake2_128)
   }
   ```

### Write remaining storage items -- 编写其它存储项

To easily track all of our kitties, we're going to standardize our logic to use a unique ID as the global key for our storage items.
This means that a single unique key will point to our Kitty object (i.e. the struct we previously declared).
为了方便跟踪所有猫咪，我们将使用唯一 ID 作为我们存储项的全局键，这样可以标准化我们的逻辑。
这意味着有一个唯一键将指向我们的 Kitty（即我们之前声明的struct）。

In order for this to work, we need to make sure that the ID for a new Kitty is always unique.
We can do this with a new storage item `Kitties` which will be a mapping from an ID (Hash) to the Kitty object.
为了让代码正确运行，我们需要确保新创建 Kitty 的 ID 始终是唯一的。
我们可以使用新的存储项“Kitties”来做到这一点，它将是从 ID（哈希）到 Kitty 对象的映射。

With this object, we can easily check for collisions by simply checking whether this storage item already contains a mapping using a particular ID.
For example, from inside a dispatchable function we could check using:
使用kitty对象，我们可以通过检查存储项里是否包含一个使用了特定ID的映射来轻松检查冲突。
例如，从可调度函数内部，我们可以使用以下命令进行检查：

```rust
ensure!(!<Kitties<T>>::exists(new_id), "This new id already exists");
```

Our runtime needs to be made aware of:
我们的runtime需要注意：

- Unique assets, like currency or Kitties (this will be held by a storage map called `Kitties`).
	独特的资产，比如货币或猫咪（名为`Kitties`的存储映射将持有它）。
- Ownership of those assets, like account IDs (this will be handled a new storage map called `KittiesOwned`).
	这些资产的所有权，例如 account IDs（名为`KittiesOwned`的新存储映射持有它）。

To create a storage instance for the `Kitty` struct, we'll be using[`StorageMap`][storage-map-kb] &mdash; a hash-map provided to us by FRAME.
我们将使用[`StorageMap`][storage-map-kb] （ FRAME 提供的一个基于hash的map结构）为`Kitty`struct创建一个存储实例。

Here's what the `Kitties` storage item looks like:
`Kitties` 存储项如下所示：

```rust
#[pallet::storage]
#[pallet::getter(fn kitties)]
pub(super) type Kitties<T: Config> = StorageMap<
	_,
	Twox64Concat,
	T::Hash,
	Kitty<T>,
>;
```

Breaking it down, we declare the storage type and assign a `StorageMap` that takes:
我们来拆解一下，我们声明了存储类型，并把它分配了一个包含如下内容的`StorageMap`：

       - The [`Twox64Concat`][2x64-rustdocs] hashing algorithm.
       - A key of type `T::Hash`.
       - A value of type `Kitty<T>`.

The `KittiesOwned` storage item is similar except that we'll be using a `BoundedVec` to keep track of some maximum number of Kitties we'll configure in `runtime/src/lib.s`.
`KittiesOwned` 存储项与上面类似，除了使用 `BoundedVec` 来跟踪我们配置在 `runtime/src/lib.s` 中 Kitties的一些最大数量。

```rust
#[pallet::storage]
#[pallet::getter(fn kitties_owned)]
/// Keeps track of what accounts own what Kitty.
pub(super) type KittiesOwned<T: Config> = StorageMap<
	_,
	Twox64Concat,
	T::AccountId,
	BoundedVec<T::Hash, T::MaxKittyOwned>,
	ValueQuery,
>;
```

Your turn! Copy the two code snippets above to replace line ACTION #7.
轮到你了！ 复制上面的两个代码片段，用他们替换 ACTION #7 行。

Before we can check our pallet compiles, we need to add a new type `MaxKittyOwned` in the config trait, which is a pallet constant type (similar to `KittyRandomness` in the previous steps).
Replace ACTION #9 with:
在我们检查pallet编译之前，我们需要在配置trait中添加一个新类型`MaxKittyOwned`，这是一个pallet常量类型（类似于前面步骤中的`KittyRandomness`）。
将 ACTION #9 行替换为：

```rust
#[pallet::constant]
type MaxKittyOwned: Get<u32>;
```

Finally, we define `MaxKittyOwned` type in `runtime/src/lib.rs`.
This is the same pattern as we followed for `Currency` and `KittyRandomness` except we'll be adding a fixed `u32` using the `parameter_types!` macro:
最后，我们将在 `runtime/src/lib.rs` 中定义 `MaxKittyOwned` 类型。
这与我们为 `Currency` 和 `KittyRandomness` 遵循的模式相同，除了使用 `parameter_types!` 宏来添加一个固定的 `u32`数值：

```rust
parameter_types! {              // <- add this macro
	// One can own at most 9,999 Kitties
	pub const MaxKittyOwned: u32 = 9999;
}

/// Configure the pallet-kitties in pallets/kitties.
impl pallet_kitties::Config for Runtime {
	type Event = Event;
	type Currency = Balances;
	type KittyRandomness = RandomnessCollectiveFlip;
	type MaxKittyOwned = MaxKittyOwned; // <- add this line
}
```

Now is a good time to check that your Kitties blockchain compiles!
现在是您检查 Kitties 区块链能否编译通过的大好时机！

```bash
cargo build --release
```

Running into difficulties? Check your solution against the [completed helper code](https://github.com/substrate-developer-hub/substrate-docs/tree/main/static/assets/tutorials/kitties-tutorial/03-dispatchables-and-events.rs) for this part of the tutorial.
遇到困难了吗？ 参考教程源代码 [completed helper code](https://github.com/substrate-developer-hub/substrate-docs/tree/main/static/assets/tutorials/kitties-tutorial/03-dispatchables-and-events.rs)，检查您的解决方案。

## Dispatchables, events, and errors -- 可调用、事件和错误

In the previous section of this tutorial, we laid down the foundations geared to manage the ownership of our Kitties &mdash; even though they don't really exist yet!
In this part we'll be putting these foundations to use by giving our pallet the ability to create a Kitty using the storage items we've declared.
Breaking things down a little, we're going to write:
在本教程的前一部分中，我们为管理 Kitties 的所有权打下了基础—— 即使它们还没有真正存在！
在这一部分中，我们将使用前一部分提供的基础功能，使用前一部分声明的存储项，来给pallet添加创建猫咪（Kitty）的功能。
稍微分解一下，我们将要编写如下代码：

- **`create_kitty`**: a dispatchable or publicly callable function allowing an account to mint
  a Kitty. 一个用来创建Kitty的可调用函数，或者直接让一个账户公开调用。
- **`mint()`**: a helper function that updates our pallet's storage items and performs error
  checks, called by `create_kitty`.一个辅助函数，用来更新pallet的存储项，执行错误检查， `create_kitty` 可以调用这个函数。
- **pallet `Events`**: using FRAME's `#[pallet::event]` attribute. 使用FRAME的`#[pallet::event]`。

At the end of this part, we'll check that everything compiles without error and call our
`create_kitty` extrinsic using the PolkadotJS Apps UI.
在这部分的最后，我们将确保代码编译没有错误，并在PolkadotJS Apps的UI中调用我们的
`create_kitty` 事务（extrinsic）功能。

<Message
  type={`yellow`}
  title={`Information`}
  text={`
If you're feeling confident, you can continue building on your codebase from the previous part.
Otherwise, refer to our starting base code at [here](https://github.com/substrate-developer-hub/substrate-docs/tree/main/static/assets/tutorials/kitties-tutorial/03-dispatchables-and-events.rs).
It also uses various "ACTION" items as a way to guide you through this section. 
如果您有信心，您可以继续在前一部分的代码基线上构建。
否则，请参考我们在 [这里](https://github.com/substrate-developer-hub/substrate-docs/tree/main/static/assets/tutorials/kitties-tutorial/03-dispatchables-and-events.rs)的启动基线代码。
它也使用各种 “ACTION” 条目作为方法来指导您完成本节。
  `}
/>

### Public and private functions -- 公共与私有函数

Before we dive right in, it's important to understand the pallet design decisions we'll be making around coding up our Kitty pallet's minting and ownership management capabilities.
在我们深入研究之前，理解我们将围绕 Kitty  pallet的铸币和所有权管理功能进行编码的pallet设计决策非常重要。

As developers, we want to make sure the code we write is efficient and elegant.
Oftentimes, optimizing for one optimizes for the other.
The way we're going to set up our pallet to optimize for both will be to break-up the "heavy lifting" logic into private helper functions.
This improves code readability and reusability too.
As we'll see, we can create private functions which can be called by multiple dispatchable functions without compromising on security.
In fact, building this way can be considered an additive security feature.
Check out [this how-to guide](/how-to-guides/v3/basics/helper-functions) about writing and using helper functions to learn more.
作为开发人员，我们希望我们编写的代码高效且优雅。
通常，优化一个地方是为了其他地方的优化。
为了优化两个地方，我们通常会设置pallet，这种方式是将一个“繁重”逻辑分解为多个私有辅助函数。
这也提高了代码的可读性和可重用性。
正如我们将看到的，我们可以创建由多个可调度函数调用的私有函数，而不会影响安全性。
事实上，以这种方式构建可以被认为是一种附加的安全功能。
想了解更多信息，可以查看关于编写和使用辅助函数的文档 [this how-to guide](/how-to-guides/v3/basics/helper-functions)。

Before jumping into implementing this approach, let's first paint the big picture of what combining dispatchables and helper functions looks like.
在开始实施这种方法之前，让我们首先描绘一下组合可调用和辅助函数的样子。

**`create_kitty`** is a dispatchable function or extrinsic that:
**`create_kitty`** 是一个可调度函数或事务：

- checks the origin is signed 检查origin是否已签名
- generates a random hash with the signing account 用签名账户生成一个随机hash值
- creates a new Kitty object using the random hash 用随机hash值创建一个新的Kitty对象
- calls a private `mint()` function 调用私有`mint()`函数

**`mint`** is a private helper function that:
**`mint`** 是一个私有辅助函数：

- checks that the Kitty doesn't already exist 检查Kitty不存在
- updates storage with the new Kitty ID (for all Kitties and for the owner's account)
	用新的Kitty ID更新存储（为所有的猫咪以及猫咪主人的账号）
- updates the new total Kitty count for storage and the new owner's account
	更新存储中的Kitty总数量与新猫咪的主人账号
- deposits an Event to signal that a Kitty has successfully been created
	发出一个 Event 以表明 Kitty 已成功创建

### Write the `create_kitty` dispatchable -- 编写可调度的 `create_kitty`

A [dispatchable][dispatchable-kb] in FRAME always follows the same structure.
All pallet dispatchables live under the `#[pallet::call]` macro which requires declaring the dispatchables section with ` impl<T: Config> Pallet<T> {}`.
Read the [documentation][frame-macros-kb] on these FRAME macros to learn how they work.
All we need to know here is that they're a useful feature of FRAME that minimizes the code required to write for pallets to be properly integrated in a Substrate chain's runtime.
FRAME 中的 [dispatchable][dispatchable-kb] 始终遵循相同的结构。
所有pallet的可调度函数都存在于`#[pallet::call]` 宏下，它需要用`impl<T: Config> Pallet<T> {}` 声明可调度函数。
阅读有关 FRAME 宏的 [文档][frame-macros-kb] 以了解它们的工作原理。
这里我们需要知道的是，这些宏是FRAME 的一个有用特性，它可以最大限度地减少为将pallet正确集成到 Substrate 链的runtime所需编写的代码。

#### Weights -- 权重

As per the requirement for `#[pallet::call]` described in its documentation, every dispatchable function must have an associated weight to it.
Weights are an important part of developing with Substrate as they provide safe-guards around the amount of computation to fit in a block at execution time.
根据其文档中描述的对 `#[pallet::call]` 的要求，每个可调度函数都必须具有关联的权重（weight）。
权重（weight）是使用 Substrate 开发的重要部分，因为它们提供了围绕计算量的安全防护，保证在生成一个区块（block）的时间内执行完毕。

[Substrate's weighting system][weights-kb] forces developers to think about the computational complexity each [extrinsic][extrinsics-kb] carries before it is called.
This allows a node to account for worst case execution time, avoiding lagging the network with extrinsics that may take longer than the specified block time.
Weights are also intimately linked to the [fee system][txn-fees-kb] for any signed extrinsic.
[Substrate's weighting system][weights-kb] 要求开发人员在调用每个 [extrinsic][extrinsics-kb] 之前仔细考虑其计算复杂度。
这允许节点考虑最坏情况的执行时间，避免因交易（extrinsic）执行所需时间超过当前区块的生成时间，这个行为会拖慢整个网络。
对于任何已签名的交易，权重也与 [fee system][txn-fees-kb] 密切相关。

As this is just a tutorial, we're going to default all weights to 100 to keep things simple.
由于这只是一个教程，我们将默认所有权重为 100 以保持简单。

Assuming you've now replaced the contents of `pallets/kitties/src/lib.rs` with [the helper file](https://github.com/substrate-developer-hub/substrate-docs/tree/main/static/assets/tutorials/kitties-tutorial/03-dispatchables-and-events.rs) for this section, find ACTION #1 and complete the beginning of the function with the lines below:
假设你现在已经用[the helper file](https://github.com/substrate-developer-hub/substrate-docs/tree/main/static/assets/tutorials/kitties-tutorial/03-dispatchables-and-events.rs) 替换了`pallets/kitties/src/lib.rs`的内容，找到 ACTION #1 并使用以下行完成函数的开头部分：

```rust
#[pallet::weight(100)]
pub fn create_kitty(origin: OriginFor<T>) -> DispatchResult {
	let sender = ensure_signed(origin)?; // <- add this line
	let kitty_id = Self::mint(&sender, None, None)?; // <- add this line
	// Logging to the console
	log::info!("A kitty is born with ID: {:?}.", kitty_id); // <- add this line

	// ACTION #4: Deposit `Created` event

	Ok(())
}
```

We won't go into [debugging](/v3/runtime/debugging/), but logging to the console is a useful tip to make sure your pallet is behaving as expected.
In order to use `log::info`, add this to your pallet's `Cargo.toml` file, using the matching version as Substrate upstream.
我们不会进入 [debugging](/v3/runtime/debugging/)，但是登录到控制台是一个有用的提示，可以确保您的pallet按预期运行。
为了使用 `log::info`，请将其添加到您的pallet的 `Cargo.toml` 文件中，使用匹配的版本作为 Substrate upstream。

### Write the `mint()` function -- 编写`mint()`函数

As seen when we wrote `create_kitty` in the previous section, we'll need to create `mint()` for writing our new unique Kitty object to the various storage items declared in Part II of this tutorial.
正如我们在上一节中编写`create_kitty` 时所看到的，我们需要创建`mint()` 来将我们新的唯一Kitty 对象写入本教程第二部分中声明的各种存储项。

Let's get right to it. Our `mint()` function will take the following arguments:
让我们开始吧。 我们的 `mint()` 函数将采用以下参数：

- **`owner`**: of type `&T::AccountId` - this indicates whom the kitty belongs to. 表明猫咪属于谁
- **`dna`**: of type `Option<[u8; 16]>` - this specifies the DNA of the kitty going to be minted. 指定要铸造猫咪的DNA
  If `None` is passed in, a random DNA will be generated.
  假如传入`None`，则会生成一个随机的DNA
- **`gender`**: of type `Option<Gender>` - ditto. 同上

And it will return `Result<T::Hash, Error<T>>`.
它会返回 `Result<T::Hash, Error<T>>`。

Paste in the following code snippet to write the `mint` function, replacing ACTION #2 in the working codebase:
粘贴以下代码片段以编写 `mint` 函数，替换代码中的 ACTION #2 行：

```rust
// Helper to mint a Kitty.
pub fn mint(
	owner: &T::AccountId,
	dna: Option<[u8; 16]>,
	gender: Option<Gender>,
) -> Result<T::Hash, Error<T>> {
	let kitty = Kitty::<T> {
		dna: dna.unwrap_or_else(Self::gen_dna),
		price: None,
		gender: gender.unwrap_or_else(Self::gen_gender),
		owner: owner.clone(),
	};

	let kitty_id = T::Hashing::hash_of(&kitty);

	// Performs this operation first as it may fail
	let new_cnt = Self::count_for_kitties().checked_add(1)
		.ok_or(<Error<T>>::CountForKittiesOverflow)?;

	// Check if the kitty does not already exist in our storage map
	ensure!(Self::kitties(&kitty_id) == None, <Error<T>>::KittyExists);

	// Performs this operation first because as it may fail
	<KittiesOwned<T>>::try_mutate(&owner, |kitty_vec| {
		kitty_vec.try_push(kitty_id)
	}).map_err(|_| <Error<T>>::ExceedMaxKittyOwned)?;

	<Kitties<T>>::insert(kitty_id, kitty);
	<CountForKitties<T>>::put(new_cnt);
	Ok(kitty_id)
}
```

Let's go over what the above code is doing.
让我们回顾一下上面的代码在做什么。

The first thing we're doing is creating a new Kitty object.
Then, we create a unique `kitty_id` using a hashing function based on the current properties of the kitty.
我们要做的第一件事是创建一个新的 Kitty 对象。
然后，我们基于猫咪的当前属性，使用一个hash函数创建一个唯一的`kitty_id`。

Next, we increment the `CountForKitties` using the storage getter function `Self::count_for_kitties()`.
We are also checking for an overflow with the `check_add()` function.
接下来，我们使用存储 getter 函数 `Self::count_for_kitties()` 来递增 `CountForKitties`。
我们还会使用 `check_add()` 函数检查溢出情况。

Our last validation is to make sure the `kitty_id` does not already exist in our `Kitties` StorageMap.
This is to avoid any possible hash key duplication insertions.
我们最后的验证是确保我们的 `Kitties` StorageMap 中不存在`kitty_id`。
这是为了避免任何可能的hash键的重复插入。

Once our checks have passed, we proceed with updating our storage items by:
一旦我们的检查通过，我们将继续通过以下方式更新我们的存储项：

1. Making use of [`try_mutate`](/rustdocs/latest/frame_support/storage/trait.StorageMap.html#tymethod.try_mutate) to update the kitty's owner vector. 利用 [`try_mutate`](/rustdocs/latest/frame_support/storage/trait.StorageMap.html#tymethod.try_mutate) 更新猫咪的所有者集合。
2. Using the [`insert`][insert-rustdocs] method provided by Substrate's StorageMap API to store the actual Kitty object and associate it with its `kitty_id`. 使用 Substrate 的 StorageMap API 提供的 [`insert`][insert-rustdocs] 方法来存储实际的 Kitty 对象并将其与它的 `kitty_id` 关联。
3. Using [`put`](/rustdocs/latest/frame_support/storage/trait.StorageValue.html#tymethod.put) provided by the StorageValue API to store the latest Kitty count. 使用 StorageValue API 提供的 [`put`](/rustdocs/latest/frame_support/storage/trait.StorageValue.html#tymethod.put) 来存储最新的 Kitty 数量。

<Message
  type="gray"
  title="A quick recap of our storage items"
  text={`
- **\`<Kitties<T>>\`**: Stores a Kitty's unique traits and price, by storing the Kitty object and
  associating it with its Kitty ID.
- **\`<KittyOwned<T>>\`**: Keeps track of what accounts own what Kitties.
- **\`<CountForKitties<T>>\`**: A count of all Kitties in existence.
`}
/>

### Implement pallet `Events` -- 实现pallet `Events`

Our pallet can also emit [Events][events-kb] at the end of the function.
This not only reports the success of a function's execution, but also tells the "off-chain world" that some particular state transition has happened.
我们的pallet还可以在函数结束时发出 [Events][events-kb]。
这不仅报告函数执行成功，还告诉“链下世界”发生了某些特定的状态转换。

FRAME helps us easily manage and declare our pallet's events using the [`#[pallet::event]`][events-rustdocs] attribute.
With FRAME macros, events are just an enum declared like this:
FRAME 使用 [`#[pallet::event]`][events-rustdocs] 属性帮助我们轻松管理和声明pallet的事件。
使用 FRAME 宏，事件只是一个像这样声明的枚举：

```rust
#[pallet::event]
#[pallet::generate_deposit(pub(super) fn deposit_event)]
pub enum Event<T: Config>{
	/// A function succeeded. [time, day]
	Success(T::Time, T::Day),
}
```

As you can see in the above snippet, we use attribute macro:
正如您在上面的代码片段中看到的，我们使用属性宏：

`#[pallet::generate_deposit(pub(super) fn deposit_event)]`

This allows us to deposit a specific event using the pattern below:
这使我们可以使用以下模式发出特定事件：

```rust
Self::deposit_event(Event::Success(var_time, var_day));
```

In order to use events inside our pallet, we need to add a new associated type `Event` inside our pallet's configuration trait `Config`.
Additionally &mdash; just as when adding any type to our pallet's `Config` trait &mdash; we also need to define it in our runtime `runtime/src/lib.rs`.
为了在pallet中使用事件，我们需要在pallet的 `Config` trait中添加一个新的关联类型`Event`。
另外，就像在我们的pallet的 `Config`  trait 中添加任何类型一样，我们还需要在runtime（`runtime/src/lib.rs`）中定义它。

This pattern is the same as when we added the `KittyRandomness` type to our pallet's configuration trait [earlier in this tutorial](#implement-on-chain-randomness) and has already been included from the initial scaffolding of our codebase:
这种模式与我们将 `KittyRandomness` 类型添加到pallet的配置trait[本教程的前面]（#implement-on-chain-randomness）时相同，并且已经包含在我们代码基线的初始脚手架中：

```rust
/// Configure the pallet by specifying the parameters and types it depends on.
#[pallet::config]
pub trait Config: frame_system::Config {
	/// Because this pallet emits events, it depends on the runtime's definition of an event.
	type Event: From<Event<Self>> + IsType<<Self as frame_system::Config>::Event>;
	//--snip--//
}
```

Declare your pallet events by replacing the ACTION #3 line with:
通过将 ACTION #3 行替换为以下内容来声明您的pallet事件：

```rust
/// A new Kitty was successfully created. \[sender, kitty_id\]
Created(T::AccountId, T::Hash),
/// Kitty price was successfully set. \[sender, kitty_id, new_price\]
PriceSet(T::AccountId, T::Hash, Option<BalanceOf<T>>),
/// A Kitty was successfully transferred. \[from, to, kitty_id\]
Transferred(T::AccountId, T::AccountId, T::Hash),
/// A Kitty was successfully bought. \[buyer, seller, kitty_id, bid_price\]
Bought(T::AccountId, T::AccountId, T::Hash, BalanceOf<T>),
```

We'll be using most of these events in the last section of this tutorial.
For now let's use the relevant event for our `create_kitty` dispatchable.
我们将在本教程的最后一节中使用这些事件中的大部分。
现在我们会为`create_kitty`调度使用相关事件。

Complete it by replacing ACTION #4 with:
将 ACTION #4 替换为：

```rust
Self::deposit_event(Event::Created(sender, kitty_id));
```

<br />
<Message
  type={`gray`}
  title={`Note`}
  text={`
If you're building your codebase from the previous part (and haven't been using the helper file
for this part) you'll need to add \`Ok(())\` and properly close the \`create_kitty\` dispatchable.
  `}
/>

### Error handling -- 错误处理

FRAME provides us with an error handling system using [`[#pallet::error]`][errors-kb] which allows us to specify errors for our pallet and use them across our pallet's functions.
FRAME 为我们提供了一个使用 [`[#pallet::error]`][errors-kb] 的错误处理系统，它允许我们为pallet指定错误，并在我们的pallet函数中使用这些错误。

Declare all possible errors using the provided FRAME macro under `#[pallet::error]`, replace line ACTION #5 with:
使用 FRAME 提供的`#[pallet::error]` 宏，声明所有可能的错误，将第 ACTION #5 行替换为：

```rust
/// Handles arithmetic overflow when incrementing the Kitty counter.
CountForKittiesOverflow,
/// An account cannot own more Kitties than `MaxKittyCount`.
ExceedMaxKittyOwned,
/// Buyer cannot be the owner.
BuyerIsKittyOwner,
/// Cannot transfer a kitty to its owner.
TransferToSelf,
/// This kitty already exists
KittyExists,
/// This kitty doesn't exist
KittyNotExist,
/// Handles checking that the Kitty is owned by the account transferring, buying or setting a price for it.
NotKittyOwner,
/// Ensures the Kitty is for sale.
KittyNotForSale,
/// Ensures that the buying price is greater than the asking price.
KittyBidPriceTooLow,
/// Ensures that an account has enough funds to purchase a Kitty.
NotEnoughBalance,
```

We'll be using these errors once we write the interactive functions in the next section.
Notice that we've already used `CountForKittiesOverflow` and `ExceedMaxKittyOwned` in our `mint` function.
一旦我们在下一节中编写完函数相互调用，我们将使用这些错误。
请注意，我们已经在 `mint` 函数中使用了 `CountForKittiesOverflow` 和 `ExceedMaxKittyOwned`。

Now's a good time to see if your chain can compile.
Instead of only checking if your pallet compiles, run the following command to see if everything can build:
这会儿是检查你的链是否可以编译的大好时机。
不要只检查您的pallet是否可以编译，而是运行以下命令以查看是否所有内容都可以构建：
```rust
cargo build --release
```

<br />
<Message
  type={`yellow`}
  title={`Information`}
  text={`
If you ran into errors, scroll to the first error message in your terminal, identify what line
is giving an error and check whether you've followed each step correctly. Sometimes a mismatch of
curly brackets will unleash a whole bunch of errors that are difficult to understand &mdash; double
check your code!
  `}
/>

Did that build fine...?
构建通过了么？

🎉**_Congratulations!_**🎉

That's the core functionality of our Kitties pallet.
In the next section you'll be able to see everything you've built so far in action.
这是我们 Kitties pallet的核心功能。
在下一部分中，您将能够看到到目前为止您所构建的所有内容。

### Testing with Polkadot-JS Apps UI -- 使用Polkadot-JS Apps UI测试

1. Run your chain and use the [PolkadotJS Apps UI](https://polkadot.js.org/apps/#/explorer) to interact with it.
   In your project directory, run:
   启动你的链，并使用[PolkadotJS Apps UI](https://polkadot.js.org/apps/#/explorer)来与链进行交互操作。
   在你的项目目录下，执行：

   ```bash
   ./target/release/node-kitties --tmp --dev
   ```

   By doing this, we're specifying to run a temporary chain in development mode, so as not to need to purge storage each time we want to start a fresh chain.
   You should be seeing block finalization happening in your terminal.
   通过这样做，我们指定在开发模式下运行一个临时链，这样每次我们想要启动一个新链时都不需要清除存储。
    您应该会在终端中看到成功出块了。

1. Head over to [Polkadot.js Apps UI][polkadotjsapps].  前往 [Polkadot.js Apps UI][polkadotjsapps]。

1. Click on the top left circular network icon, open the "Development" section, and choose "Local Node".
   Your node is default to be `127.0.0.1.:9944`.
   单击左上角的圆形网络图标，打开“Development”部分，然后选择“Local Node”。
    您的节点默认是 `127.0.0.1.:9944`。

1. Go to: _"Developer"_ -> _"Extrinsics"_ and submit a signed extrinsic using _substrateKitties_ by calling the `createKitty` dispatchable.
   Make 3 different transactions from Alice, Bob and Charlie's accounts.
   转到：_"Developer"_ -> _"Extrinsics"_ 并通过调用 `createKitty` 可调度项使用 _substrateKitties_ 提交签名的交易。
    从 Alice、Bob 和 Charlie 的账户进行 3 次不同的交易。

1. Check for the associated event _"Created"_ by going to "_Network_" -> "_Explorer_".
   You should be able to see the events emitted and query their block details.
   通过转到“_Network_”->“_Explorer_”检查相关事件_"Created"_。
    您应该能够看到发出的事件并查询它们的区块的详细信息。
    
1. Check your newly created Kitty's details by going to "_Developer_" -> "_Chain State_".
   Select the _substrateKitties_ pallet and query `Kitties(Hash): Kitty`.
   通过转到“_Developer_”->“_Chain State_”检查您新创建的 Kitty 的详细信息。
    选择 _substrateKitties_  pallet 并查询 `Kitties(Hash): Kitty`。

   Be sure to uncheck the "include option" box and you should be able to see the details of your newly minted Kitty in the following format:
   请务必取消选中“include option”框，您应该能够以以下格式查看新铸造的 Kitty 的详细信息：

   ```json
   substrateKitties.kitties: Option<Kitty>
   [
     [
       [
         0x15cb95604033af239640125a30c45b671a282f3ef42c6fc48a78eb18464b30a9
       ],
       {
         dna: 0xaf2f2b3f77e110a56933903a38cde1eb,
         price: null,
         gender: Female,
         owner: 5GrwvaEF5zXb26Fz9rcQpDWS57CtERHpNehXCPcNoHGKutQY
       }
     ]
   ]
   ```

1. Check that other storage items correctly reflect the creation of additional Kitties.
	检查其他存储项是否正确反映了其他猫咪的创建。

## Interacting with your Kitties -- 与你的Kitties应用交互

Up until this point in the tutorial, you've built a chain capable of only creating and tracking the ownership of Kitties.
Now that that's done, we want to make our runtime more like a game by introducing other functions like buying and selling Kitties.
In order to achieve this, we'll first need to enable users to mark and update the price of their Kitties.
Then we can add functionality to enable users to transfer, buy, and breed Kitties.
到目前为止，您已经构建了一个只能创建和跟踪 Kitties 所有权的链。
现在已经完成了，我们希望通过引入其他功能（例如购买和出售 Kitty）来使我们的runtime更像游戏。
为了实现这一点，我们首先需要让用户能够标记和更新他们猫咪的价格。
然后我们可以添加功能，使用户能够转移、购买和繁殖猫咪。

### Set a price for each Kitty -- 为每只猫咪设置价格

In [the helper file for this part of the tutorial](https://github.com/substrate-developer-hub/substrate-docs/tree/main/static/assets/tutorials/kitties-tutorial/04-interacting-functions.rs), you'll notice that the structure of `set_price` is already laid out.
在 [这部分教程的辅助文件](https://github.com/substrate-developer-hub/substrate-docs/tree/main/static/assets/tutorials/kitties-tutorial/04-interacting-functions.rs)中，你会注意到 `set_price` 的结构已经布局好了。

Your job is to replace ACTION lines #1a, #1b, #2 and #3 with what you'll learn in sections A-D below.
你的工作是用你将在下面的 A-D 部分学习的内容替换 ACTION 行 #1a、#1b、#2 和 #3。

#### A. Checking Kitty owner -- 检查Kitty所有者

As we create functions which modify objects in storage, we should always check first that only the appropriate users can successfully execute the logic in these dispatchable functions.
当我们创建存储中对象的修改函数时，我们应该始终首先检查是否只有正确的用户才能成功执行这些可调度函数中的逻辑。

The general pattern for an ownership check will look something like this:
所有权检查的一般模式如下所示：

```rust
let owner = Self::owner_of(object_id).ok_or("No owner for this object")?;

ensure!(owner == sender, "You are not the owner");
```

The first line checks if `Self::owner_of(object_id)` return a `Some(val)`. If yes, it is
transformed into `Result::Ok(val)`, and finally extract `val` out from `Result`. If not, it is
transformed into `Result::Err()` with provided error message, and return early with the error
object.
第一行检查 `Self::owner_of(object_id)` 是否返回 `Some(val)`。 如果是，那就是转化为`Result::Ok(val)`，最后从`Result`中提取`val`。 如果不是，那就是转换为 `Result::Err()` 并提供错误信息，并提前返回错误信息目的。

The second line checks if `owner == sender`. If true, the program execution continues to the next
line. If not, `Result::Err("You are not the owner")` error object is immediately returned from the
function.
第二行检查是否 `owner == sender`。 如果是真，则程序执行到下一行。 如果不是，则立即返回`Result::Err("You are not the owner")` 错误对象。

**Your turn!**
**轮到你了！**

Paste in this code snippet to replace ACTION #1a:
粘贴下列代码片段去替换 ACTION #1a：

```rust
ensure!(Self::is_kitty_owner(&kitty_id, &sender)?, <Error<T>>::NotKittyOwner);
```

Paste the following in ACTION #1b:
复制ACTION #1b里的代码：

```rust
pub fn is_kitty_owner(kitty_id: &T::Hash, acct: &T::AccountId) -> Result<bool, Error<T>> {
	match Self::kitties(kitty_id) {
		Some(kitty) => Ok(kitty.owner == *acct),
		None => Err(<Error<T>>::KittyNotExist)
	}
}
```

The line pasted in ACTION #1b is actually combining two checks together.
In case `Self::is_kitty_owner()` returns an error object `Err(<Error<T>>::KittyNotExist)`, it is returned early with `<Error<T>>::KittyNotExist` by the `?`.
If it returns `Ok(bool_val)`, the `bool_val` is extracted, and if false, returns `<Error<T>>::NotKittyOwner` error.
ACTION #1b 中粘贴的行实际上是将两个检查组合在一起。
如果 `Self::is_kitty_owner()` 返回一个错误对象 `Err(<Error<T>>::KittyNotExist)`，它会被 `?` 提前返回 `<Error<T>>::KittyNotExist` .
如果返回 `Ok(bool_val)`，则提取 `bool_val`，如果bool_val为 false，则返回 `<Error<T>>::NotKittyOwner` 错误。

#### B. Updating the price of our Kitty object -- 更新Kitty对象的价格

Every Kitty object has a price attribute that we've set to `None` as a default value inside the `mint` function [earlier in this tutorial](#write-the-mint-function):
每个 Kitty 对象都有一个 price 属性，我们在 `mint` 函数 [本教程的前面部分](#write-the-mint-function)中将其默认值设置为 `None` 

```rust
let kitty = Kitty::<T> {
	dna: dna.unwrap_or_else(Self::gen_dna),
	price: None,                           //<-- 👀 here
	gender: gender.unwrap_or_else(Self::gen_gender),
	owner: owner.clone(),
};
```

To update the price of a Kitty, we'll need to:
为了更新Kitty的价格，我们需要：

- Get the Kitty object in storage. 从存储中获取Kitty对象。
- Update the object with the new price. 用一个新价格更新Kitty对象。
- Save it back into storage. 把Kitty对象存回存储。

Changing a value in an existing object in storage would be written in the following way:
更改存储中某个现有对象的值，需要按以下方式编写：

```rust
let mut object = Self::get_object(object_id);
object.value = new_value;

<Object<T>>::insert(object_id, object);
```

<br />
<Message
  type="gray"
  title="Note"
  text={`
Rust expects you to declare a variable as mutable (using the \`mut\` keyword) whenever its value is
going to be updated. Rust希望你将一个变量声明为可变的（用\`mut\`关键字），这样无论何时你都可以变更这个变量的值。
`}
/>

**Your turn!**
**轮到你了！**

Paste in the following snippet to replace the ACTION #2 line:
复制下列代码去替换ACTION #2行：

```rust
kitty.price = new_price.clone();
<Kitties<T>>::insert(&kitty_id, kitty);
```

#### D. Deposit an Event -- 发出一个事件

Once all checks are passed and the new price is written to storage, we can deposit an event [just like we did before](#implement-pallet-events).
Replace the line marked as ACTION #3 with:
一旦所有检查都通过并且新价格被写入存储，我们就可以发出一个事件[就像我们之前所做的那样](#implement-pallet-events)。
将标记为 ACTION #3 的行替换为：

```rust
// Deposit a "PriceSet" event.
Self::deposit_event(Event::PriceSet(sender, kitty_id, new_price));
```

Now whenever the `set_price` dispatchable is called successfully, it will emit a `PriceSet` event.
现在，无论何时可调度函数 `set_price`被成功调用，它都会发出 `PriceSet` 事件。

### Transfer a Kitty -- 转移Kitty

Based on the `create_kitty` function we built earlier, you already have the tools and knowledge you'll need to create the transfer functionality.
The main difference is that there are **two parts** to achieving this:
基于我们之前构建的“create_kitty”函数，您已经拥有创建传输功能所需的工具和知识。
实现此目标的主要区别有**两个部分**：

1. A **dispatchable function** called `transfer()`: this is a publicly callable dispatchable exposed by your pallet. 可调度函数`transfer()`：这是一个被pallet暴露出来的可以公开调用的可调度函数。
2. A **private helper function** called `transfer_kitty_to()`: this will be a private helper function called by `transfer()` to handle all storage updates when transferring a Kitty. 私有辅助函数`transfer_kitty_to()`：这是一个私有辅助函数，在转移Kitty时，`transfer()`可以调用这个函数来执行更新操作。

Separating the logic this way makes the private `transfer_kitty_to()` function reusable by other dispatchable functions of our pallet without needing to duplicate code.
In our case, we're going to reuse it for the `buy_kitty` dispatchable we'll be creating next.
以这种方式分离逻辑，使得私有 `transfer_kitty_to()` 函数可以被我们pallet的其他可调度函数重用，而无需复制代码。
在我们的例子中，我们将在接下来要创建的 `buy_kitty` 可调度函数中重用这个函数。

#### `transfer` -- `转移`

Paste in the following snippet to replace ACTION #4 in the template code:
粘贴以下代码段以替换模板代码中的 ACTION #4：

```rust
#[pallet::weight(100)]
pub fn transfer(
	origin: OriginFor<T>,
	to: T::AccountId,
	kitty_id: T::Hash
) -> DispatchResult {
	let from = ensure_signed(origin)?;

	// Ensure the kitty exists and is called by the kitty owner
	ensure!(Self::is_kitty_owner(&kitty_id, &from)?, <Error<T>>::NotKittyOwner);

	// Verify the kitty is not transferring back to its owner.
	ensure!(from != to, <Error<T>>::TransferToSelf);

	// Verify the recipient has the capacity to receive one more kitty
	let to_owned = <KittiesOwned<T>>::get(&to);
	ensure!((to_owned.len() as u32) < T::MaxKittyOwned::get(), <Error<T>>::ExceedMaxKittyOwned);

	Self::transfer_kitty_to(&kitty_id, &to)?;

	Self::deposit_event(Event::Transferred(from, to, kitty_id));

	Ok(())
}
```

By now the above pattern should be familiar.
We always check that the transaction is signed; then we verify that:
到目前为止，我们应该很熟悉上述模式了。
我们始终检查交易是否已签署； 然后我们验证：

1. The Kitty being transferred is owned by the sender of this transaction. 被转移的 Kitty 属于交易的发送者。
2. The Kitty is not transferred to its owner (a redundant operation). Kitty 不可以转移给它的所有者（这是一个冗余操作）。
3. The recipient has the capacity to receive one more kitty. 接受者有再接收一只猫咪的容量。

Lastly we call the `transfer_kitty_to` helper to update all storage items appropriately.
最后，我们调用 `transfer_kitty_to` 助手函数来更新所有存储项。

#### `transfer_kitty_to`

The `transfer_kitty_to` function will be a helper to perform all storage updates once a Kitty is transferred (and it is going to be called when a kitty is bought and sold too).
All it needs to do is perform safety checks and update the following storage items:
转移猫咪后，`transfer_kitty_to` 函数将成为执行所有存储更新的助手函数（并且在购买和出售猫咪时也会调用它）。
它需要做的就是执行安全检查，并更新以下存储项：

- `KittiesOwned`: to update the owner of the Kitty. 更新Kitty的所有者。
- `Kitties`: to reset the price in the Kitty object to `None`. 重置Kitty对象中价格为`None`

Copy the following to replace ACTION #5:
复制以下内容来替换 ACTION #5：

```rust
#[transactional]
pub fn transfer_kitty_to(
	kitty_id: &T::Hash,
	to: &T::AccountId,
) -> Result<(), Error<T>> {
	let mut kitty = Self::kitties(&kitty_id).ok_or(<Error<T>>::KittyNotExist)?;

	let prev_owner = kitty.owner.clone();

	// Remove `kitty_id` from the KittiesOwned vector of `prev_owner`
	<KittiesOwned<T>>::try_mutate(&prev_owner, |owned| {
		if let Some(ind) = owned.iter().position(|&id| id == *kitty_id) {
			owned.swap_remove(ind);
			return Ok(());
		}
		Err(())
	}).map_err(|_| <Error<T>>::KittyNotExist)?;

	// Update the kitty owner
	kitty.owner = to.clone();
	// Reset the ask price so the kitty is not for sale until `set_price()` is called
	// by the current owner.
	kitty.price = None;

	<Kitties<T>>::insert(kitty_id, kitty);

	<KittiesOwned<T>>::try_mutate(to, |vec| {
		vec.try_push(*kitty_id)
	}).map_err(|_| <Error<T>>::ExceedMaxKittyOwned)?;

	Ok(())
}
```

Notice the use of [`#[transactional]`](/rustdocs/latest/frame_support/attr.transactional.html) which we imported at the very beginning of this tutorial.
It allows us to write dispatchable functions that commit changes to the storage only if the annotated function returns `Ok`.
Otherwise all changes are discarded.
请注意我们在本教程开始时导入的 [`#[transactional]`](/rustdocs/latest/frame_support/attr.transactional.html) 的使用。
它允许我们编写可调度函数，仅当带注释的函数返回“Ok”时，才会更新存储。
否则，所有更改都将被丢弃。

### Buy a Kitty -- 购买Kitty

We'll need to ensure two things before we can allow the user of this function to purchase a Kitty:
在允许用户使用此功能购买 Kitty 之前，我们需要确保两件事：

- Check that the Kitty is for sale. - 检查猫咪是否正在出售。
- Check whether the Kitty's current price is within the user's budget and whether the user has enough free balance. 检查Kitty的当前价格是否在用户的预算之内，以及用户是否有充足的余额。

Replace line ACTION #6 to check whether a Kitty is for sale:
替换 ACTION #6 行以检查 Kitty 是否在售：

```rust
// Check the kitty is for sale and the kitty ask price <= bid_price
if let Some(ask_price) = kitty.price {
	ensure!(ask_price <= bid_price, <Error<T>>::KittyBidPriceTooLow);
} else {
	Err(<Error<T>>::KittyNotForSale)?;
}

// Check the buyer has enough free balance
ensure!(T::Currency::free_balance(&buyer) >= bid_price, <Error<T>>::NotEnoughBalance);
```

In a similar vein, we have to verify whether the user has the capacity to receive a Kitty.
Remember we're using a [`BoundedVec`](/rustdocs/latest/frame_support/storage/bounded_vec/struct.BoundedVec.html) that can only hold a fixed number of Kitties, defined in our pallet's `MaxKittyOwned` constant.
Replace ACTION #7 with:
同样，我们必须验证用户是否有足够的容量来接收 Kitty。
请记住，我们使用的 [`BoundedVec`](/rustdocs/latest/frame_support/storage/bounded_vec/struct.BoundedVec.html) 只能容纳固定数量的猫咪，我们用pallet的 `MaxKittyOwned` 常量定义了`BoundedVec`类型。
将 ACTION #7 替换为：

```rust
// Verify the buyer has the capacity to receive one more kitty
let to_owned = <KittiesOwned<T>>::get(&buyer);
ensure!((to_owned.len() as u32) < T::MaxKittyOwned::get(), <Error<T>>::ExceedMaxKittyOwned);

let seller = kitty.owner.clone();
```

We'll use [FRAME's Currency trait][currency-frame-rustdocs] to adjust account balances using its [`transfer` method][transfer-currency-rustdocs].
It's useful to understand why it's important to use the `transfer` method in particular and how we'll be accessing it:
我们将使用 [FRAME的货币（Currency）trait][currency-frame-rustdocs] 的 [`transfer` 方法][transfer-currency-rustdocs]来调整账户余额。
为什么 `transfer` 方法很重要，以及如何访问它，理解这些是很有用的：

- The reason we'll be using it is to ensure our runtime has the same understanding of currency throughout the pallets it interacts with.
  The way that we ensure this is to use the `Currency` trait given to us by `frame_support`.
  我们将使用它的原因是，确保我们的runtime，以及runtime中所有pallet对货币有相同的理解。
  我们确保这一点的方法是使用 `frame_support` 提供给我们的 `Currency` trait。

- Conveniently, it handles a [`Balance`][currency-balances-rustdocs] type, making it compatible with `BalanceOf` type we created for `kitty.price`.
  Take a look at how the `transfer` function we'll be using [is structured][currency-transfer-rustdocs]:
  很方便，它处理 [`Balance`][currency-balances-rustdocs] 类型，使其与我们为 `kitty.price` 创建的 `BalanceOf` 类型兼容。
   看看我们将使用的 `transfer` 函数是如何[结构化][currency-transfer-rustdocs]的：

  ```rust
  fn transfer(
  	source: &AccountId,
  	dest: &AccountId,
  	value: Self::Balance,
  	existence_requirement: ExistenceRequirement
  ) -> DispatchResult
  ```

Now we can make use of the `Currency` type in our pallet's `Config` trait and `ExistenceRequirement` that we [initially started with in the first section](#write-out-pallet_kitties-scaffold).
这会儿我们可以使用[本教程第一部分开头](#write-out-pallet_kitties-scaffold)中pallet的 `Config` trait中的`Currency` 类型和 `ExistenceRequirement`。

Update the balances of both the caller of this function and the receiver, replacing ACTION #8:
更新此函数的调用者和接收者的余额，替换 ACTION #8：

```rust
// Transfer the amount from buyer to seller
T::Currency::transfer(&buyer, &seller, bid_price, ExistenceRequirement::KeepAlive)?;

// Transfer the kitty from seller to buyer
Self::transfer_kitty_to(&kitty_id, &buyer)?;

// Deposit relevant Event
Self::deposit_event(Event::Bought(buyer, seller, kitty_id, bid_price));
```

<br />
<Message
  type="gray"
  title="Note"
  text={`
Both of the above operations, \`T::Currency::transfer()\`, and \`Self::transfer_kitty_to()\` could
fail which is why we check for the returned result in each case. If \`Err\` is returned, we also return from the
function immediately. In order to keep the storage consistent with these potential changes, we also annotate this function as
\`#[transactional]\`.
上述两种操作，\`T::Currency::transfer()\` 和 \`Self::transfer_kitty_to()\` 都可能失败，这就是为什么我们在每种情况下都会检查返回结果。 如果返回了\`Err\`，我们将立即从函数返回。 为了让存储与这些潜在变更保持一致，我们还将用\`#[transactional]\`标注函数。
`}
/>

### Breed Kitties -- 繁殖 Kitty

The logic behind breeding two Kitties is to multiply each corresponding DNA segment from two Kitties, which will produce a new DNA sequence.
Then, that DNA is used when minting a new Kitty.
This helper function is already provided for you in the template file for this section.
两只猫咪繁殖小猫咪的背后逻辑是将两只猫咪的每个相应的 DNA 片段相乘，这将产生一个新的 DNA 序列。
然后，在铸造新猫咪时使用该 DNA。
这个帮助函数已经给你放在本节的模板文件中。

Paste in the following to complete the `breed_kitty` function, replacing line ACTION #9:
复制下列内容替换 ACTION #9行，以便完成 `breed_kitty` 函数

```rust
let new_dna = Self::breed_dna(&parent1, &parent2)?;
```

Don't forget to add `breed_dna(&parent1, &parent2)` helper function (peep it's definition in [the helper file](https://github.com/substrate-developer-hub/substrate-docs/blob/main/static/assets/tutorials/kitties-tutorial/04-interacting-functions.rs#L227)).
不要忘记添加 `breed_dna(&parent1, &parent2)` 辅助函数（定义在[辅助文件中](https://github.com/substrate-developer-hub/substrate-docs/blob/main/static/assets/tutorials/kitties-tutorial/04-interacting-functions.rs#L227)）

Now that we've used the user inputs of Kitty IDs and combined them to create a new unique Kitty ID, we can use the `mint()` function to write that new Kitty to storage.
Replace line ACTION #10 to complete the `breed_kitty` extrinsic:
现在我们已经使用了 Kitty ID 的用户输入，并将它们组合起来创建了一个新的唯一 Kitty ID，我们可以使用 `mint()` 函数将一个新 Kitty 写入存储。
替换 ACTION #10，以便完成 `breed_kitty` 事务：

```rust
Self::mint(&sender, Some(new_dna), None)?;
```

### Genesis configuration -- 配置创世（Genesis）信息

The final step before our pallet is ready to be used is to set the genesis state of our storage items.
We'll make use of FRAME's `#[pallet::genesis_config]` to do this.
Essentially, this allows us to declare what the Kitties object in storage contains in the genesis block.
在pallet能使用之前，还有最后一步，设置我们存储项的创世状态。
我们将使用 FRAME 的 `#[pallet::genesis_config]` 来执行此操作。
本质上，这允许我们在创世区块中声明存储中的 Kitties 对象可以包含那些内容。

Copy the following code to replace ACTION #11:
复制以下代码以替换 ACTION #11：

```rust
// Our pallet's genesis configuration.
#[pallet::genesis_config]
pub struct GenesisConfig<T: Config> {
	pub kitties: Vec<(T::AccountId, [u8; 16], Gender)>,
}

// Required to implement default for GenesisConfig.
#[cfg(feature = "std")]
impl<T: Config> Default for GenesisConfig<T> {
	fn default() -> GenesisConfig<T> {
		GenesisConfig { kitties: vec![] }
	}
}

#[pallet::genesis_build]
impl<T: Config> GenesisBuild<T> for GenesisConfig<T> {
	fn build(&self) {
		// When building a kitty from genesis config, we require the dna and gender to be supplied.
		for (acct, dna, gender) in &self.kitties {
			let _ = <Pallet<T>>::mint(acct, Some(dna.clone()), Some(gender.clone()));
		}
	}
}
```

To let our chain know about our pallet's genesis configuration, we need to modify the `chain_spec.rs` file in our project's `node` folder.
It's important you make sure you use the name of the pallet instance in `runtime/src/lib.rs`, which in our case was `SubstrateKitties`.
Go to `node/src/chain_spec.rs`, add `use node_kitties_runtime::SubstrateKittiesConfig;` at the top of the file and add the following snippet inside the `testnet_genesis` function:

```rust
//-- snip --
substrate_kitties: SubstrateKittiesConfig {
	kitties: vec![],
},
//-- snip --
```

### Build, run and interact with your Kitties -- 构建、运行并与你的Kitties应用交互

If you've completed all of the preceding parts and steps of this tutorial, you're ready to run your chain and start interacting with all the new capabilities of your Kitties pallet!
如果您已完成本教程的所有前面内容和步骤，您就可以启动您的区块链，并开始与 Kitties pallet的所有新功能进行交互了！

Build and run your chain using the following commands:
使用以下命令构建并运行您的区块链：

```bash
cargo build --release
./target/release/node-kitties --dev --tmp
```

Now check your work using the Polkadot-JS Apps just like [we did previously](#testing-with-polkadot-js-apps-ui).
Once your chain is running and connected to the PolkadotJS Apps UI, perform these manual checks:
现在使用 Polkadot-JS 应用程序检查您的工作，就像 [我们之前所做的](#testing-with-polkadot-js-apps-ui)。
一旦您的区块链运行并连接到 PolkadotJS Apps UI，请执行以下手动检查：

- Fund multiple users with tokens so they can all participate 给多个用户充值token，这样他们都可以参与
- Have each user create multiple Kitties 让每个用户都创建多个猫咪
- Try to transfer a Kitty from one user to another using the right and wrong owner 试着用正确与错误的所有者将 Kitty 从一个用户转移到另一个用户
- Try to set the price of a Kitty using the right and wrong owner 试着用正确与错误的所有者来设置猫咪的价格
- Buy a Kitty using an owner and another user 用猫咪所有者与其他用户购买猫咪
- Use too little funds to purchase a Kitty 使用很少（低于价格）的资金购买猫咪
- Overspend on the cost of the Kitty and ensure that the balance is reduced appropriately 使用高价（超过猫咪价格）购买猫咪，并确保余额被准确减少
- Breed a Kitty and check that the new DNA is a mix of the old and new 繁殖一只猫咪，并检查新 DNA 是否是新旧猫咪 DNA 的混合体

After all of these actions, confirm that all users have the correct number of Kitties; that the total Kitty count is correct; and any other storage variables are correctly represented.
完成所有这些操作后，确认所有用户拥有正确数量的猫咪； 猫咪总数是正确的； 并且任何其他存储变量都能正确展示。

🎉**_Congratulations!_**🎉

You've successfully created the backend of a fully functional Substrate chain capable of creating and managing Substrate Kitties.
The basic capabilities of our Kitties application could also be abstracted to other NFT-like use cases.
Most importantly, at this point in the tutorial you should have all the knowledge you need to start creating your own pallet logic and dispatchable functions.
您已经成功创建了一个功能齐全的 Substrate 链的后端，这个区块链能够创建和管理猫咪。
我们的 Kitties 应用程序的基本功能也可以抽象为其他类似 NFT 的用例。
最重要的是，到这里，通过本教程您应该已经具备创建您自己pallet的逻辑和可调度函数所需知识。

## Next steps -- 下一步

Move on to [Part II](/tutorials/v3/kitties/pt2/) to connect your chain to the front-end template and create a user interface to visualize and interact with your Kitties!
继续[Part II](/tutorials/v3/kitties/pt2/)教程，将您的区块链连接到前端模板，并创建一个可以与之交互的用户界面，来可视化您的 Kitties！

[transfer-currency-rustdocs]: https://crates.parity.io/frame_support/traits/tokens/currency/trait.Currency.html#tymethod.transfer
[frame-balances-rustdocs]: https://crates.parity.io/frame_support/traits/tokens/currency/trait.Currency.html
[polkadotjs-ui]: https://polkadot.js.org/apps/#/explorer
[rust-u8]: https://doc.rust-lang.org/std/primitive.u8.html
[currency-frame-rustdocs]: /rustdocs/latest/frame_support/traits/tokens/currency/index.html
[currency-balances-rustdocs]: /rustdocs/latest/frame_support/traits/tokens/currency/trait.Currency.html#associatedtype.Balance
[balances-frame]: /rustdocs/latest/pallet_balances/index.html
[currency-transfer-rustdocs]: /rustdocs/latest/frame_support/traits/tokens/currency/trait.Currency.html#tymethod.transfer
[installation]: /v3/getting-started/installation
[substrate-node-template]: https://github.com/substrate-developer-hub/substrate-node-template
[pallets-kb]: /v3/runtime/frame#pallets
[macros-kb]: /v3/runtime/macros#frame-macros-and-attributes
[storagevalue-rustdocs]: /rustdocs/latest/frame_support/storage/trait.StorageValue.html
[storage-api-rustdocs]: /rustdocs/latest/frame_support/storage/index.html
[template-code]: https://github.com/substrate-developer-hub/substrate-how-to-guides/tree/main/static/code/kitties-tutorial
[runtime-kb]: /v3/concepts/runtime
[kickstart-tool]: https://github.com/Keats/kickstart
[storage-macro-kb]: /v3/runtime/macros#palletstorage
[default-rustdocs]: https://doc.rust-lang.org/std/default/trait.Default.html
[randomness-rustdocs]: /rustdocs/latest/frame_support/traits/trait.Randomness.html
[hash-rustdocs]: /rustdocs/latest/sp_runtime/traits/trait.Hash.html
[blake2-rustdocs]: /rustdocs/latest/sp_core/hashing/fn.blake2_128.html
[randomness-collective-flip-frame]: /rustdocs/latest/pallet_randomness_collective_flip/index.html
[nonce-rustdocs]: /rustdocs/latest/frame_system/struct.AccountInfo.html#structfield.nonce
[2x64-rustdocs]: /rustdocs/latest/frame_support/struct.Twox64Concat.html
[prelude-traits-rustdocs]: /rustdocs/latest/sp_std/prelude/index.html#traits
[derive-macro-rust]: https://doc.rust-lang.org/reference/procedural-macros.html#derive-macros
[storage-best-practice-kb]: /v3/runtime/storage#best-practices
[storage-map-kb]: /v3/runtime/storage/#storage-map
[storage-value-kb]: /v3/runtime/storage#storage-value
[currency-frame]: /rustdocs/latest/frame_support/traits/tokens/currency/trait.Currency.html#associatedtype.Balance
[frame-macros-kb]: /v3/runtime/macros#palletcall
[txn-fees-kb]: /v3/runtime/weights-and-fees
[weights-kb]: /v3/concepts/weight
[contains-key-rustdocs]: /rustdocs/latest/frame_support/storage/trait.StorageMap.html#tymethod.contains_key
[insert-rustdocs]: /rustdocs/latest/frame_support/storage/trait.StorageMap.html#tymethod.insert
[storage-value-rustdocs]: /rustdocs/latest/frame_support/storage/types/struct.StorageValue.html#method.put
[storagemap-rustdocs]: /rustdocs/latest/frame_support/storage/types/struct.StorageMap.html#method.insert
[events-rustdocs]: /rustdocs/latest/frame_support/attr.pallet.html#event-palletevent-optional
[events-kb]: /v3/runtime/events-and-errors
[polkadotjsapps]: https://polkadot.js.org/apps/#/explorer
[dispatchresult-rustdocs]: /rustdocs/latest/frame_support/dispatch/type.DispatchResult.html
[dispatchable-kb]: /v3/getting-started/glossary#dispatch
[extrinsics-kb]: /v3/concepts/execution#executing-extrinsics
[errors-kb]: /v3/runtime/events-and-errors
