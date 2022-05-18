![kitties-tutorial.png](https://img-blog.csdnimg.cn/img_convert/2c5c9cf44ba2bbb3157208827aaed8ee.png)
欢迎来到Substrate Kitties课程。
本课程将向您介绍如何构建一个可以创建并持有非同质化代币（NFT，这个NFT名称为Substrate Kitties）的区块链。课程分为部2部分：

Part I 描述如何构建Kitties pallet，以及这个pallet怎样与你所创建的Substrate Kitties应用进行交互。

[Part II](/tutorials/v3/kitties/pt2) 描述如何开发一个前端，这个前端需要与Part I 的Substrate Kitties区块链进行交互。

## 教程目标
- 学习构建并运行Substrate节点的一些基本模式。
- 写一个自定义FRAME pallet，并与你的节点runtime集成。
- 学习如何创建与更新存储项(items)。
- 编写pallet外部交易（extrinsics）与辅助函数。
- Use the PolkadotJS API to connect a Substrate node to a custom a front-end. 用Polkadot JS API把自定义的前端与Substrate节点进行连接。

本教程假设您已经在您的机器上安装了使用 Substrate 构建的先决条件。
通过安装 [Rust and the Rust toolchain](/v3/getting-started/installation) 确保您已经为 Substrate 开发配置好了环境

### 要构建的内容

我们尽量让内容简单一些，方便您以后可以决定如何改进您的 Substrate Kitties 链。
为了达到我们的构建目的，Kitties应用只做以下事情：

- 通过一些初始资源或者使用已有猫咪（Kitties）来繁殖新猫咪。

- 所有者可以设定一个价格来出售猫咪。

- 所有者可以把猫咪转让给别人。

### 不会涉及的内容
本教程不会包括以下内容：
- 为pallet编写测试用例。
- 使用正确的权重（weight）数值。

完成本课程后，您可以参考 [how-to guides](/how-to-guides/v3) 来了解如何在应用中集成以上内容。

定好自己的学习目标，按照自己的节奏，参考教程的每个步骤 ，最好的方法就是自己不停地试验！

在从一个步骤移动到下一个步骤前，请确保您的pallet构建没有任何错误。
使用[模板文件](https://github.com/substrate-developer-hub/substrate-docs/tree/main/static/assets/tutorials/kitties-tutorial)辅助你完成每一个部分。

Most of the code changes will be in `/pallets/kitties/src/lib.rs`.
如果你被某个问题卡住了，可以参考 [Substrate note template仓库的\`tutorials/solutions/kitties\`分支](https://github.com/substrate-developer-hub/substrate-node-template/tree/tutorials/solutions/kitties)上的完整源代码。
大部分代码修改都是在 `/pallets/kitties/src/lib.rs` 文件中。

## 基本设置

在我们开始构建Kitties应用前，我们首先需要做一些基础工作。
这部分介绍了一些使用 Substrate node template 的基本模式，这些模式涉及自定义pallet的设置与如何包含一个简单的存储项。

### 设定你的template node

[Substrate node template][substrate-node-template] 为我们提供了一个可定制的区块链节点，包括内置的网络层和共识层。
我们需要关注的是构建我们的 [runtime][runtime-kb] 和 [pallets][pallets-kb] 的逻辑。

首先，我们需要设置我们的项目名称和依赖项。
我们将使用一个名为 [kickstart][kickstart-tool] 的 命令行（CLI） 工具来重命名我们的node template。

1. 通过运行 `cargo install kickstart` 来安装。

1. 安装好 `kickstart` 后，在工作空间的根目录下运行下列命令：

  	```bash
   kickstart https://github.com/sacha-l/kickstart-substrate
   ```

   这个命令将克隆出最新的node template代码，并询问您希望如何调用node与pallet。

1. 输入：

   - `kitties` - 这是节点（node）名称， 这个节点将被命名为“node-kitties”。
   - `kitties` - 这是pallet名称。 这个pallet将被命名为“pallet-kitties”。

   这将创建一个名为 `kitties` 的目录，目录中包含 [Substrate node template][substrate-node-template] 的副本，副本中包含了与template node、runtime与pallet对应的名称修改。

1. 用您喜欢的代码编辑器，打开`kitties`目录并将其重命名为`kitties-tutorial`（或任何您喜欢的名称），以帮助您保持工作井井有条。

   注意 `kickstart` 命令修改的目录：

   - **/node/** - 此文件夹包含node与runtime、node与RPC 客户端的所有交互逻辑。
   - **/pallets/** - 这里是您所有自定义pallet的所在地。
   - **/runtime/** - This folder is where all pallets (both custom "internal" and "external" ones) are aggregated and implemented for the chain's runtime.
	**/runtime/** - 所有pallet（包含自定义的“内部”和“外部”）聚合在此文件夹，并实现为区块链的runtime。

1. 在 `runtime/src/lib.rs` 中，您还会注意到我们修改后的模板pallet名称的实例仍然包含 `TemplateModule`。
 
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

### 编写 `pallet_kitties` 脚手架代码

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

> 您可以先删除 `benchmarking.rs`、`mock.rs` 和 `tests.rs`。**我们不会在本教程中学习如何使用这些。** 如果您想了解测试的工作原理，请查看 [this how-to guide](/how-to-guides/v3/testing/transfer-function)。

Substrate 中的 [Pallets][pallets-kb] 用于定义运行时（runtime）逻辑。
在这个例子中，我们将创建一个独立的pallet来管理我们的 Substrate Kitties 应用程序的所有逻辑。

请注意，pallet目录 `/pallets/kitties/` 与pallet名称是不同的。
按照Cargo的理念，我们的pallet名称是“pallet-kitties”。

通过 `pallets/kitties/src/lib.rs`文件中的内容，我们可以勾勒出我们自己pallet的基础结构。

每个FRAME pallet包含：

- `frame_support` 和 `frame_system` 依赖项。
- 必需的 [attribute macros][macros-kb]（即配置trait、存储项和函数调用）。

<Message
  type={`yellow`}
  title={`信息`}
  text={`
随着本教程进行到下一部分时，我们将更新其他依赖项。
  `}
/>

这将是我们在本教程中构建的最简单 pallet 版本。
这是本教程下一部分添加代码的起点。
就像本教程的 [辅助文件](https://github.com/substrate-developer-hub/substrate-docs/tree/main/static/assets/tutorials/kitties-tutorial)，它包含了标有**TODO**的注释（ 表示我们稍后将完成代码）与**ACTION**（表示将在当前编写代码）注释。

1. 粘贴以下代码到文件`/pallets/kitties/src/lib.rs`：

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

2. 注意pallet中正在使用`sp_io`文件。
   确保在你pallet项目的`Cargo.toml`文件中，声明了这个依赖项：

   ```toml
   sp-io = { default-features = false, git = "https://github.com/paritytech/substrate.git", branch = "polkadot-v0.9.19" }
   ```

3. 现在尝试运行以下命令来构建您的pallet。 暂时还不需要构建整个区块链，因为我们还没有在runtime中实现`Currency`类型。
    到此为止，我们可以来检查pallet中是否有错误：

   ```bash
   cargo build -p pallet-kitties
   ```

你会注意到 Rust 编译器会给你关于未使用的导入警告。
没关系！
忽略它们吧—— 我们将在教程的后面部分使用这些导入。

### 添加存储项

让我们向runtime添加一个最简单的逻辑：在runtime中存储一个变量的函数。
为此，我们将使用Substrate提供的 [存储 API][storage-api-rustdocs]：[`StorageValue`][storagevalue-rustdocs]，这个API是一个依赖[storage macro][storage-macro-kb]的trait。

就我们的目的而言，这意味着对于我们要声明的任何存储项，我们必须事先包含`#[pallet::storage]`宏。
了解有关声明存储项的更多信息 [这里](/v3/runtime/storage#declaring-storage-items)。

用以下内容替换`pallets/kitties/src/lib.rs`文件中的ACTION行：

```rust
#[pallet::storage]
#[pallet::getter(fn count_for_kitties)]
/// Keeps track of the number of Kitties in existence.
pub(super) type CountForKitties<T: Config> = StorageValue<_, u64, ValueQuery>;
```

这为我们的pallet创建了一个存储项，用以跟踪现有猫咪的总数。

### 添加货币实现

在继续构建node之前，我们需要将 Currency 类型添加到pallet的runtime实现中。 在 `runtime/src/lib.rs` 中，添加以下内容：

```rust
impl pallet_kitties::Config for Runtime {
	type Event = Event;
	type Currency = Balances; // <-- 添加这行
}
```

现在构建您的node，并确保您没有遇到任何错误。 假如是第一次构建，这将需要一些时间的。

```bash
cargo build --release
```

🎉 **_恭喜您!_** 🎉

您已经完成了这个系列的第一部分。
在这个阶段，您学习了以下各种模式：

- 自定义 Substrate node template与自定义pallet。
- 构建 Substrate 链，并检查pallet是否能通过编译。
- 声明单个值 \`u64\` 存储项。

## 唯一性、自定义类型和存储映射

您已经深入了解了使用 FRAME （**一个把各种模块化实体聚合为runtime的框架**）来开发pallet的一些关键概念，包括编写一个 [存储struct](/how-to-guides/v3/pallet-design/storage-value) and [实现随机数的trait](/how-to-guides/v3/pallet-design/randomness)。

使用[辅助代码](https://github.com/substrate-developer-hub/substrate-docs/tree/main/static/assets/tutorials/kitties-tutorial/02-create-kitties.rs) 帮助您完成每一步。
这将是接下来几个步骤的基础。

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

除了这段代码，我们还需要导入 `serde`。
将其添加到您pallet的 Cargo.toml 文件中，使用匹配的版本作为最新的 Substrate。

### 编写Kitty Struct的框架代码

Rust 中的 Struct 是一种有用的构造，有助于存储具有共性的数据。
出于我们的目的，我们的 Kitty 将携带多个属性，我们可以将它们存储在单个结构中，而不是每个属性使用单独的存储项。
这在尝试优化存储读取和写入时会派上用场，因此我们的runtime可以执行更少的读取/写入来更新多个值。
阅读更多关于存储最佳实践的信息 [这里][storage-best-practice-kb]。

#### 包含的信息

我们先来看看单个 Kitty 会包括哪些信息：

- **`dna`**：用于识别 Kitty 的 DNA 的哈希值，与它的独特特征相对应。
   DNA 还用于繁殖新的猫咪并跟踪不同的猫咪世代。
- **`price`**：这是购买 Kitty 所需的金额或者所有者设定的售出价格。
- **`gender`**：一个可以是 `Male` 或 `Female` 的枚举。
- **`owner`**：指定单个所有者的account ID。

#### Sketching out the types held by our struct -- 预估我们的结构所持有的类型

从上节内容中结构体中的条目，我们可以推断出以下类型：

- **`[u8; 16]`** for `dna` - 使用 16 个字节来表示猫咪的 DNA。
- **`BalanceOf`** for `price` - 这是一个使用 FRAME 中 [`Currency` trait][currency-frame] 的自定义类型。
- **`Gender`** for `gender` - 我们需要创建这个类型！

首先，在声明结构之前，我们需要为 `BalanceOf` 和 `AccountOf` 添加自定义类型。
用以下代码替换ACTION #1：

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
  title="注意"
  text={`
我们定义了\`<BalanceOf<T>>\` 和 \`AccountOf<T>\` 类型，并在 \`Kitty\` 中使用它们。如果你想知道 Rust 中的第一行是什么意思，它是定义一个类型别名 \`AccountOf<T>\`，它是指向 trait \`frame_system::Config\` 的关联类型 \`AccountId\` 的速写，这个类型需要绑定泛型\`T\`。\n\n更多关于这种类型的语法在 [the Rust book](https://doc.rust-lang.org/book/ch19-03-advanced-traits.html#specifying-placeholder-types-in-trait-definitions-with-associated-types)。
`}
/>

为了使用我们的结构（strutct），需要注意我们如何使用 `derive` 宏来包含 [各种辅助trait][prelude-traits-rustdocs] 。
我们需要添加 `TypeInfo` 以使我们的结构可以访问此 trait。
在你的pallet顶部添加以下内容：

```rust
use scale_info::TypeInfo;
```

对于 `Gender`类型，我们需要定义自己的枚举类型和辅助函数。

### 为 `Gender` 编写自定义类型

我们刚刚创建了一个结构，它需要一个名为 `Gender` 的自定义类型。
这种类型将是我们定义猫咪性别的枚举类型。
要创建它，您将构建以下部分：

- **enum声明**，指定 `Male` 和 `Female` 值。
- **为我们的 Kitty 结构实现一个辅助函数**。

1. #### 声明自定义枚举

   用下列的枚举声明替换ACTION #2:

   ```rust
   #[derive(Clone, Encode, Decode, PartialEq, RuntimeDebug, TypeInfo, MaxEncodedLen)]
   #[cfg_attr(feature = "std", derive(Serialize, Deserialize))]
   pub enum Gender {
   	Male,
   	Female,
   }
   ```

	注意在枚举声明，必须使用 [derive macro][derive-macro-rust]。
    这将我们的枚举包装在数据结构中，它需要在我们的runtime中与其他类型进行交互。
    为了使用 `Serialize` 和 `Deserialize`，您需要在 `pallets/kitties/Cargo.toml` 中添加 `serde` crate，使用匹配的版本作为Substrate upstream。

    太好了，我们现在知道如何创建自定义结构了。
    但是如何提供一种方法给 Kitty 结构设置一个性别值呢？
    为此，我们需要再学习一件事。

#### 为Kitty结构实现个辅助函数

   Configuring a struct is useful in order to pre-define a value in our struct.
   For example, when setting a value in relation to what another function returns.
   In our case we have a similar situation where we need to configure our Kitty struct in such a way that sets `Gender` according to a Kitty's DNA.
	为了在我们的结构中预定义一个值，配置一个结构是很有用的。
    例如，在我们赋一个值时，对应有另外一个函数可以返回该值。
    在我们的例子中，我们有类似的情况，我们需要配置 Kitty 结构，以便根据 Kitty 的 DNA 值来给`Gender`赋值。

   我们只会在[创建 Kitty](#write-the-create_kitty-dispatchable) 时使用这个函数。
    无论如何，让我们现在就学习如何编写，并完成这个函数。
    我们将创建一个名为 `gen_gender` 的公共函数来返回 `Gender` 类型，使用随机函数来选择`Gender` 枚举值。

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

   现在，任何时候在我们的pallet中调用 `gen_gender()` 时，它都会返回基于伪随机确定的 `Gender` 枚举值。

### 实现链上随机性

如果我们希望能够区分这些猫咪，我们需要开始赋予它们某些独特的属性！
在上一步中，我们使用了尚未实际定义的“KittyRandomness”。
让我们开始吧。

我们将使用 `frame_support` 中的 [Randomness trait][randomness-rustdocs] 来完成此功能。
它能够生成一个随机种子，我们将用这个种子来创建独特的猫咪，并繁殖新的猫咪。

1. 在您的pallet的配置trait中，定义一个受 `Randomness` trait 约束的新类型。

   需要用参数指定来自`frame_support` 中的 `Randomness` trait，以替换 `Output` 和 `BlockNumber` 泛型。
    看一下 [文档][randomness-rustdocs] 和源代码实现以了解其工作原理。
    为了达到目的，我们希望使用这个trait的函数输出的是 [`Blake2 128-bit hash`][blake2-rustdocs] ，你应该会注意到在现存代码的头部你已经声明了这些内容。
    
   用下列代码替换ACTION #5行：
   ```rust
   type KittyRandomness: Randomness<Self::Hash, Self::BlockNumber>;
   ```

1. 指定runtime里的实际类型。
	
   由于我们在pallet配置中添加了一个新类型，所以我们需要配置runtime，来设置这个新类型的具体类型。
    如果我们想更改`KittyRandomness` 正在使用的算法，而不需要在pallet里修改它的使用位置，这可能会派上用场。

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
   看看这个实现随机性 [how-to-guide](/how-to-guides/v3/pallet-design/randomness) ，以防您遇到困难。
   
1. 生成随机DNA

   生成 DNA 类似于使用随机性来随机分配性别类型。
   不同之处在于我们将使用在前一部分中导入的 `blake2_128`。
   我们还将使用 `frame_system` pallet中的 [extrinsic_index](https://docs.rs/frame-system/latest/frame_system/pallet/struct.Pallet.html#method.extrinsic_index) ，为了生成不同的哈希值，我们可以在同一个块中多次调用这个函数。
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

### 编写剩余存储项

为了方便跟踪所有猫咪，我们将使用唯一 ID 作为我们存储项的全局键，这样可以标准化我们的逻辑。
这意味着有一个唯一键将指向我们的 Kitty（即我们之前声明的struct）。

为了让代码正确运行，我们需要确保新创建 Kitty 的 ID 始终是唯一的。
我们可以使用新的存储项“Kitties”来做到这一点，它将是从 ID（哈希）到 Kitty 对象的映射。

使用kitty对象，我们可以通过检查存储项里是否包含一个使用了特定ID的映射来轻松检查冲突。
例如，从可调度函数内部，我们可以使用以下命令进行检查：

```rust
ensure!(!<Kitties<T>>::exists(new_id), "This new id already exists");
```

我们的runtime需要注意：

- 独特的资产，比如货币或猫咪（名为`Kitties`的存储映射将持有它）。
- 这些资产的所有权，例如 account IDs（名为`KittiesOwned`的新存储映射持有它）。

我们将使用[`StorageMap`][storage-map-kb] （ FRAME 提供的一个基于hash的map结构）为`Kitty`结构创建一个存储实例。

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

我们来拆解一下，我们声明了存储类型，并把它分配了一个包含如下内容的`StorageMap`：

       - [`Twox64Concat`][2x64-rustdocs] 的hash算法。
       - 一个`T::Hash`类型的key值。
       - 一个`Kitty<T>`类型的value值。

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

轮到你了！ 复制上面的两个代码片段，用他们替换 ACTION #7 行。

在我们检查pallet能否编译之前，我们需要在配置trait中添加一个新类型`MaxKittyOwned`，这是一个pallet常量类型（类似于前面步骤中的`KittyRandomness`）。
将 ACTION #9 行替换为：

```rust
#[pallet::constant]
type MaxKittyOwned: Get<u32>;
```

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

现在是您检查 Kitties 区块链能否编译通过的大好时机！

```bash
cargo build --release
```

遇到困难了吗？ 参考教程 [已完成的辅助代码](https://github.com/substrate-developer-hub/substrate-docs/tree/main/static/assets/tutorials/kitties-tutorial/03-dispatchables-and-events.rs)，检查您的解决方案。

## 可调用、事件和错误

在本教程的前一部分中，我们为管理 Kitties 的所有权打下了基础—— 即使它们还没有真正存在！
在这一部分中，我们将使用前一部分提供的基础功能，使用前一部分声明的存储项，来给pallet添加创建猫咪（Kitty）的功能。
稍微分解一下，我们将要编写如下代码：

- **`create_kitty`**: 一个用来铸造Kitty的可调用函数，或者用一个账户直接公开调用。
- **`mint()`**: 这是一个辅助函数，用来更新pallet的存储项，执行错误检查， `create_kitty` 可以调用这个函数。
- **pallet `Events`**: 使用FRAME的 `#[pallet::event]` 属性。

在这部分的最后，我们将确保代码编译没有错误，并在Polkadot JS Apps的UI中调用我们的 `create_kitty` 外部事务（extrinsic）功能。

<Message
  type={`yellow`}
  title={`信息`}
  text={`
如果您有信心，您可以继续在前一部分的代码基线上构建。
否则，请参考我们在 [这里](https://github.com/substrate-developer-hub/substrate-docs/tree/main/static/assets/tutorials/kitties-tutorial/03-dispatchables-and-events.rs)的启动基线代码。
它也使用各种 “ACTION” 条目作为方法来指导您完成本节。
  `}
/>

### 公有与私有函数

Before we dive right in, it's important to understand the pallet design decisions we'll be making around coding up our Kitty pallet's minting and ownership management capabilities.
在我们深入研究之前，我们将围绕 Kitty  pallet的铸币和所有权管理功能进行编码，理解Kitty pallet的这些设计决策非常重要。

作为开发人员，我们希望我们编写的代码高效且优雅。
通常，优化一个地方是为了其他地方的优化。
为了优化两个地方，我们通常会设置pallet，这种方式是将一个“繁重”逻辑分解为多个私有辅助函数。
这也提高了代码的可读性和可重用性。
正如我们将看到的，我们可以创建由多个可调度函数调用的私有函数，而不会影响安全性。
事实上，以这种方式构建可以被认为是一种附加的安全功能。
想了解更多信息，可以查看关于编写和使用辅助函数的文档 [this how-to guide](/how-to-guides/v3/basics/helper-functions)。

Before jumping into implementing this approach, let's first paint the big picture of what combining dispatchables and helper functions looks like.
在开始实施这种方法之前，让我们首先描绘一下把可调用和辅助函数组合起来是什么样子。

**`create_kitty`** 是一个可调度函数或外部事务（extrinsic）：

- 检查origin是否已签名
- 用签名账户生成一个随机hash值
- 用随机hash值创建一个新的Kitty对象
- 调用私有`mint()`函数

**`mint`** 是一个私有辅助函数：

- 检查当前猫咪并不存在
- 用当前新猫咪ID更新存储（为所有的猫咪以及猫咪主人的账号）
- 更新存储中的猫咪总数与新猫咪的主人账号
- 发出一个 Event 以表明 Kitty 已成功创建

### 编写可调度的 `create_kitty`

FRAME 中的 [dispatchable][dispatchable-kb] 始终遵循相同的结构。
所有pallet的可调度函数都存在于`#[pallet::call]` 宏下，它需要用`impl<T: Config> Pallet<T> {}` 声明可调度函数。
阅读有关 FRAME 宏的 [文档][frame-macros-kb] 以了解它们的工作原理。
这里我们需要知道的是，这些宏是FRAME 的一个有用特性，它可以最大限度地减少将pallet正确集成到 Substrate 链的runtime所需编写的代码。

#### 权重（Weights）

根据其文档中描述的对 `#[pallet::call]` 的要求，每个可调度函数都必须具有关联的权重（weight）。
权重（weight）是使用 Substrate 开发的重要部分，因为它们提供了围绕计算量的安全防护，保证在生成一个区块（block）的时间内执行完毕。

[Substrate的称重系统][weights-kb] 要求开发人员在调用每个 [extrinsic][extrinsics-kb] 之前仔细考虑其计算复杂度。
这允许节点考虑最坏情况的执行时间，避免因外部交易（extrinsic）执行所需时间超过当前区块的生成时间，这个行为会拖慢整个网络。
对于任何已签名的外部交易（extrinsic），权重也与 [计费系统（fee system）][txn-fees-kb] 密切相关。

由于这只是一个教程，我们为保持简单，默认将所有权重设置为 100。

假设你现在已经用[辅助文件](https://github.com/substrate-developer-hub/substrate-docs/tree/main/static/assets/tutorials/kitties-tutorial/03-dispatchables-and-events.rs) 替换了`pallets/kitties/src/lib.rs`的内容，找到 ACTION #1 并使用以下行完成函数的开头部分：

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

我们不会进入 [debugging](/v3/runtime/debugging/)，但是登录到控制台是一个有用的提示，可以确保您的pallet按预期运行。
为了使用 `log::info`，请将其添加到您的pallet的 `Cargo.toml` 文件中，使用匹配的版本作为 Substrate upstream。

### 编写`mint()`函数

正如我们在编写上一节的 `create_kitty` 时所看到的，我们需要创建`mint()` 来将我们新的唯一 Kitty 对象写入本教程第二部分中声明的各种存储项。

让我们开始吧。 我们的 `mint()` 函数将采用以下参数：

- **`owner`**: `&T::AccountId`类型 - 表明猫咪属于谁。
- **`dna`**: `Option<[u8; 16]>`类型 - 指定要铸造猫咪的DNA。
  假如传入`None`，则会生成一个随机的DNA
- **`gender`**: `Option<Gender>`类型 - 同上

它会返回 `Result<T::Hash, Error<T>>`。

粘贴以下代码片段来编写 `mint` 函数，替换代码中的 ACTION #2 行：

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

让我们回顾一下上面的代码在做什么。

我们要做的第一件事是创建一个新的 Kitty 对象。
然后，我们基于猫咪的所有属性，使用一个hash函数创建一个唯一的`kitty_id`。

接下来，我们使用存储 getter 函数 `Self::count_for_kitties()` 来递增 `CountForKitties`。
我们还会使用 `check_add()` 函数检查溢出情况。

最后的验证是确保我们的 `Kitties` StorageMap 中不存在`kitty_id`。
这是为了避免hash键重复插入的可能性。

一旦我们的检查通过，我们将继续通过以下方式更新我们的存储项：

1. 利用 [`try_mutate`](/rustdocs/latest/frame_support/storage/trait.StorageMap.html#tymethod.try_mutate) 更新猫咪的所有者集合。
1. 使用 Substrate 的 StorageMap API 提供的 [`insert`][insert-rustdocs] 方法来存储实际的 Kitty 对象，并将Kitty对象与其 `kitty_id` 关联。
1. 使用 StorageValue API 提供的 [`put`](/rustdocs/latest/frame_support/storage/trait.StorageValue.html#tymethod.put) 方法来存储最新的 Kitty 数量。

<Message
  type="gray"
  title="快速回顾一下我们的所有存储项"
  text={`
- **\`<Kitties<T>>\`**: 通过存储Kitty对象及其Kitty ID，来存储Kitty独一无二的特性（trait）与价格。
- **\`<KittyOwned<T>>\`**: 跟踪持有Kitty的所有账号。
- **\`<CountForKitties<T>>\`**: 所有现存Kitty的总数。
`}
/>

### 实现pallet `Events`

我们的pallet还可以在函数结束时发出 [Events][events-kb]。
这不仅报告函数执行成功，还起到把链上发生的特定状态转换告诉“链外世界”。

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

正如您在上面的代码片段中看到的，我们使用了属性宏：

`#[pallet::generate_deposit(pub(super) fn deposit_event)]`

允许我们使用以下模式发出某个事件：

```rust
Self::deposit_event(Event::Success(var_time, var_day));
```

为了在pallet中使用事件，我们需要在pallet的 `Config` trait中添加一个新的关联类型`Event`。
另外，就像在我们的pallet的 `Config`  trait 中添加任何类型一样，我们还需要在runtime（`runtime/src/lib.rs`）中定义它。

这种模式与我们将 `KittyRandomness` 类型添加到pallet的配置trait[本教程的前面]（#implement-on-chain-randomness）时相同，并且已经包含在我们代码基线的初始代码框架中：

```rust
/// Configure the pallet by specifying the parameters and types it depends on.
#[pallet::config]
pub trait Config: frame_system::Config {
	/// Because this pallet emits events, it depends on the runtime's definition of an event.
	type Event: From<Event<Self>> + IsType<<Self as frame_system::Config>::Event>;
	//--snip--//
}
```

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

我们将在本教程的最后一节中使用这些事件中的大部分。
现在我们会为`create_kitty`调度使用相关事件。

将 ACTION #4 替换为：

```rust
Self::deposit_event(Event::Created(sender, kitty_id));
```

<br />
<Message
  type={`gray`}
  title={`注意`}
  text={`
If you're building your codebase from the previous part (and haven't been using the helper file
for this part) you'll need to add \`Ok(())\` and properly close the \`create_kitty\` dispatchable.
假如你正在从前一部分（这部分的辅助文件还未使用）来构建你的代码基线，你需要添加\`Ok(())\`来正确结束可调度的 \`create_kitty\`
  `}
/>

### 错误处理

FRAME 为我们提供了一个使用 [`[#pallet::error]`][errors-kb] 的错误处理系统，它允许我们为pallet指定错误，并在我们的pallet函数中使用这些错误。

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

一旦我们在下一节中编写完函数相互调用，我们将使用这些错误。
请注意，我们已经在 `mint` 函数中使用了 `CountForKittiesOverflow` 和 `ExceedMaxKittyOwned`。

这会儿是检查你的链是否可以编译的大好时机。
不要只检查您的pallet是否可以编译，而是运行以下命令以查看是否所有内容都可以构建：
```rust
cargo build --release
```

<br />
<Message
  type={`yellow`}
  title={`信息`}
  text={`
假如你遇到错误，滚动你的终端，定位到发生第一个错误的地方，辨认发生错误的行，检查你是否遵守了每一个步骤。有时一个不匹配的花括号都会引发一大堆难以理解的错误。切记要仔细检查你的代码！
  `}
/>

构建通过了么？

🎉 **_恭喜您!_** 🎉

这是我们 Kitties pallet的核心功能。
在下一部分中，您将能够看到到目前为止您所构建的所有内容。

### 使用Polkadot-JS Apps UI测试

1. 启动你的链，并使用[PolkadotJS Apps UI](https://polkadot.js.org/apps/#/explorer)来与链进行交互操作。
   在你的项目目录下，执行：

   ```bash
   ./target/release/node-kitties --tmp --dev
   ```

   通过这样做，我们指定在开发模式下运行一个临时链，这样每次我们想要启动一个新链时都不需要清除存储。
    您应该会在终端中看到成功出块了。

1. 前往 [Polkadot.js Apps UI][polkadotjsapps]。

1. 单击左上角的圆形网络图标，打开“Development”部分，然后选择“Local Node”。
    您的节点默认是 `127.0.0.1.:9944`。

1. 转到：“_Developer_” ->“ _Extrinsics_” 并通过调用 `createKitty` 可调度项使用 _substrateKitties_ 提交签名的外部交易（extrinsic）。
    从 Alice、Bob 和 Charlie 的账户进行 3 次不同的交易。

1. 通过转到“_Network_”->“_Explorer_”检查相关的“_Created_”事件。
    您应该能够看到发出的事件，并查询它们的区块详细信息。
    
1. 通过转到“_Developer_”->“_Chain State_”检查您新创建的 Kitty 的详细信息。
    选择 _substrateKitties_  pallet 并查询 `Kitties(Hash): Kitty`。

   请务必不要选中“include option”框，您应该能够以以下格式查看新铸造的 Kitty 的详细信息：

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

1. 检查其他存储项是否正确反映了其他猫咪的创建。

## 与你的Kitties应用交互

Up until this point in the tutorial, you've built a chain capable of only creating and tracking the ownership of Kitties.
Now that that's done, we want to make our runtime more like a game by introducing other functions like buying and selling Kitties.
In order to achieve this, we'll first need to enable users to mark and update the price of their Kitties.
Then we can add functionality to enable users to transfer, buy, and breed Kitties.
到目前为止，您构建了一条仅能创建和跟踪 Kitties 所有权的链。
既然已经完成了已有的功能，现在我们希望通过引入其他功能（例如购买和出售 Kitty）来使我们的runtime更像游戏。
为了实现这一点，我们首先需要让用户能够标记和更新他们猫咪的价格。
然后我们可以添加功能，使用户能够转移、购买和繁殖猫咪。

### 为每只猫咪设定价格

在 [这个教程的辅助文件](https://github.com/substrate-developer-hub/substrate-docs/tree/main/static/assets/tutorials/kitties-tutorial/04-interacting-functions.rs)中，你会注意到 `set_price` 的结构已经排好了。

你的工作是用你将在下面的 A-D 部分学习的内容替换 ACTION 行 #1a、#1b、#2 和 #3。

#### A. 检查Kitty所有者

As we create functions which modify objects in storage, we should always check first that only the appropriate users can successfully execute the logic in these dispatchable functions.
当我们给存储中对象的添加一个修改函数时，我们应该首先检查只有正确的用户才能成功执行这些可调度函数。

所有权检查的常用模式如下所示：

```rust
let owner = Self::owner_of(object_id).ok_or("No owner for this object")?;

ensure!(owner == sender, "You are not the owner");
```

第一行检查 `Self::owner_of(object_id)` 是否返回 `Some(val)`。 如果是，那就是转化为`Result::Ok(val)`，最后从`Result`中提取`val`。 如果不是，那就是转换为 `Result::Err()` 并提供错误信息，并提前返回一个错误信息对象。

第二行检查是否 `owner == sender`。 如果是真，则程序执行到下一行。 如果不是，则立即返回`Result::Err("You are not the owner")` 错误对象。

**轮到你了！**

粘贴下列代码片段去替换 ACTION #1a：

```rust
ensure!(Self::is_kitty_owner(&kitty_id, &sender)?, <Error<T>>::NotKittyOwner);
```

复制ACTION #1b里的代码：

```rust
pub fn is_kitty_owner(kitty_id: &T::Hash, acct: &T::AccountId) -> Result<bool, Error<T>> {
	match Self::kitties(kitty_id) {
		Some(kitty) => Ok(kitty.owner == *acct),
		None => Err(<Error<T>>::KittyNotExist)
	}
}
```

ACTION #1b 中粘贴的行实际上是将两个检查组合在一起。
如果 `Self::is_kitty_owner()` 返回一个错误对象 `Err(<Error<T>>::KittyNotExist)`，通过使用 `?` 提前返回 `<Error<T>>::KittyNotExist` .
如果返回 `Ok(bool_val)`，则提取 `bool_val`，如果bool_val为 false，则返回 `<Error<T>>::NotKittyOwner` 错误。

#### B. 更新Kitty对象的价格

每个 Kitty 对象都有一个 price 属性，我们在 `mint` 函数 [本教程的前面部分](#write-the-mint-function)中将其默认值设置为 `None` ：

```rust
let kitty = Kitty::<T> {
	dna: dna.unwrap_or_else(Self::gen_dna),
	price: None,                           //<-- 👀 here
	gender: gender.unwrap_or_else(Self::gen_gender),
	owner: owner.clone(),
};
```

为了更新Kitty的价格，我们需要：

- 从存储中获取Kitty对象。
- 用一个新价格来更新Kitty对象。
- 把Kitty对象存回存储。

更改存储中某个现有对象的值，需要按以下方式编写：

```rust
let mut object = Self::get_object(object_id);
object.value = new_value;

<Object<T>>::insert(object_id, object);
```

<br />
<Message
  type="gray"
  title="注意"
  text={`
Rust希望你将一个变量声明为可变的（用\`mut\`关键字），这样无论何时你都可以变更这个变量的值。
`}
/>

**轮到你了！**

复制下列代码去替换ACTION #2行：

```rust
kitty.price = new_price.clone();
<Kitties<T>>::insert(&kitty_id, kitty);
```

#### D. 发出事件

一旦所有检查都通过并且新价格被写入存储，我们就可以发出一个事件[就像我们之前所做的那样](#implement-pallet-events)。
将 ACTION #3 行替换为：

```rust
// Deposit a "PriceSet" event.
Self::deposit_event(Event::PriceSet(sender, kitty_id, new_price));
```

现在，无论何时可调度函数 `set_price`被成功调用，它都会发出 `PriceSet` 事件。

### 转移Kitty

基于我们之前构建的`create_kitty`函数，您已经拥有创建转移函数（transfer）所需的工具和知识。
实现此目标的主要区别有**两个部分**：

1. **可调度（dispatchable）函数**`transfer()`：这是一个被pallet暴露出来的可以公开调用的函数。
1. **私有辅助函数** `transfer_kitty_to()`：这是一个私有辅助函数，在转移Kitty时，`transfer()`可以调用这个函数来执行存储的更新操作。

以这种方式分离逻辑，使得私有 `transfer_kitty_to()` 函数可以被我们pallet的其他可调度函数重用，而无需复制代码。
在我们的例子中，我们将在接下来要创建的 `buy_kitty` 可调度函数中重用这个函数。

#### `转移`

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

到目前为止，我们应该很熟悉上述模式了。
我们始终检查交易是否已签名， 然后我们验证：

1. 被转移的 Kitty 属于交易的发送者。
1. 不可以把 Kitty 转移给本人（这是一个冗余操作）。
1. 确保接受者有再接收一只猫咪的容量。

最后，我们调用 `transfer_kitty_to` 助手函数来更新对应的所有存储项。

#### `transfer_kitty_to`

转移猫咪后，`transfer_kitty_to` 函数将成为执行所有存储更新的助手函数（并且在购买和出售猫咪时也会调用它）。
它需要做的就是执行安全检查，并更新以下存储项：

- `KittiesOwned`: 更新Kitty的所有者。
- `Kitties`: 重置Kitty对象的价格为`None`

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

请注意我们在本教程开始时导入的 [`#[transactional]`](/rustdocs/latest/frame_support/attr.transactional.html) 的使用。
它允许我们编写可调度函数，仅当带注释的函数返回“Ok”时，才会更新存储。
否则，所有更改都将被丢弃。

### 购买Kitty

在允许用户使用此函数购买 Kitty 之前，我们需要确保两件事：

- 检查Kitty是否正在出售。
- 检查Kitty的当前价格是否在用户的预算之内，以及用户是否有充足的余额。

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

类似地，我们必须验证用户是否有足够的容量来接收 Kitty。
请记住，我们使用的 [`BoundedVec`](/rustdocs/latest/frame_support/storage/bounded_vec/struct.BoundedVec.html) 只能容纳固定数量的猫咪，我们用pallet的 `MaxKittyOwned` 常量定义了`BoundedVec`类型。
将 ACTION #7 替换为：

```rust
// Verify the buyer has the capacity to receive one more kitty
let to_owned = <KittiesOwned<T>>::get(&buyer);
ensure!((to_owned.len() as u32) < T::MaxKittyOwned::get(), <Error<T>>::ExceedMaxKittyOwned);

let seller = kitty.owner.clone();
```

我们将使用 [FRAME的货币（Currency）trait][currency-frame-rustdocs] 的 [`transfer` 方法][transfer-currency-rustdocs]来调整账户余额。
为什么 `transfer` 方法很重要，以及如何访问它，理解这些是很有用的：

- 我们将使用它的原因是，确保我们的runtime，以及runtime中所有pallet对货币有相同的理解。
  我们确保这一点的方法是使用 `frame_support` 提供给我们的 `Currency` trait。

- 很方便，它处理 [`Balance`][currency-balances-rustdocs] 类型，使其与我们为 `kitty.price` 创建的 `BalanceOf` 类型兼容。
   看看我们将使用的 `transfer` 函数是如何[结构化][currency-transfer-rustdocs]的：

  ```rust
  fn transfer(
  	source: &AccountId,
  	dest: &AccountId,
  	value: Self::Balance,
  	existence_requirement: ExistenceRequirement
  ) -> DispatchResult
  ```

这会儿我们可以使用[本教程第一部分开头](#write-out-pallet_kitties-scaffold)中pallet的 `Config` trait中的`Currency` 类型和 `ExistenceRequirement`。

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
  title="注意"
  text={`
上述两种操作，\`T::Currency::transfer()\` 和 \`Self::transfer_kitty_to()\` 都可能失败，这就是为什么我们在每种情况下都会检查返回结果。 如果返回了\`Err\`，我们将立即从函数返回。 为了让存储与这些潜在变更保持一致，我们还将用\`#[transactional]\`标注函数。
`}
/>

### 繁殖 Kitty

两只猫咪繁殖小猫咪的背后逻辑是将两只猫咪的每个相应的 DNA 片段相乘，这将产生一个新的 DNA 序列。
然后，在铸造新猫咪时使用该 DNA。
这个辅助函数已经给你放在本节的模板文件中。

复制下列内容替换 ACTION #9行，以便完成 `breed_kitty` 函数

```rust
let new_dna = Self::breed_dna(&parent1, &parent2)?;
```

不要忘记添加 `breed_dna(&parent1, &parent2)` 辅助函数（定义在[辅助文件中](https://github.com/substrate-developer-hub/substrate-docs/blob/main/static/assets/tutorials/kitties-tutorial/04-interacting-functions.rs#L227)）

现在我们已经使用了 Kitty ID 的用户输入，并将它们组合起来创建了一个新的唯一 Kitty ID，我们可以使用 `mint()` 函数将一个新 Kitty 写入存储。
替换 ACTION #10，以便完成 `breed_kitty` 外部交易（extrinsic）：

```rust
Self::mint(&sender, Some(new_dna), None)?;
```

### 配置创世（Genesis）信息

在pallet能使用之前，还有最后一步，设置我们存储项的创世（genesis）状态。
我们将使用 FRAME 的 `#[pallet::genesis_config]` 来执行此操作。
本质上，这允许我们在创世区块中声明存储中的 Kitties 对象可以包含那些内容。

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

为了让我们的链知道pallet的创世配置，我们需要修改项目的 `node` 文件夹中的 `chain_spec.rs` 文件。
确保 `runtime/src/lib.rs` 中的pallet实例的名称是`SubstrateKitties`，这一个很重要。
跳到 `node/src/chain_spec.rs` 文件，在文件顶部添加`use node_kitties_runtime::SubstrateKittiesConfig;`，并在`testnet_genesis`函数中添加以下代码段：

```rust
//-- snip --
substrate_kitties: SubstrateKittiesConfig {
	kitties: vec![],
},
//-- snip --
```

### 构建、运行并与你的Kitties应用交互

如果您已完成本教程的所有前面内容和步骤，您就可以启动您的区块链，并开始与 Kitties pallet的所有新功能进行交互了！

使用以下命令构建并运行您的区块链：

```bash
cargo build --release
./target/release/node-kitties --dev --tmp
```

现在使用 Polkadot-JS 应用程序检查您的工作，就像 [我们之前所做的](#testing-with-polkadot-js-apps-ui)。
一旦您的区块链运行并连接到 PolkadotJS Apps UI，请执行以下手动检查：

- 找到持有token的账户，因为只有持有token的账户才可以使用。
- 给每个用户都创建多个猫咪
- 试着用正确与错误的所有者将 Kitty 从一个用户转移到另一个用户
- 试着用正确与错误的所有者来设置猫咪的价格
- 用猫咪所有者或者其他用户来购买猫咪
- 使用很少（低于价格）的资金购买猫咪
- 使用高价（超过猫咪价格）购买猫咪，并确保余额被准确减少
- 繁殖一只猫咪，并检查新 DNA 是否是新旧猫咪 DNA 的混合体

完成所有这些操作后，确认所有用户拥有正确数量的猫咪； 猫咪总数是正确的； 并且任何其他存储变量都能正确展示。

🎉 **_恭喜您!_** 🎉

您已经成功创建了一个功能齐全的 Substrate 链的后端，这个区块链能够创建和管理猫咪。
我们的 Kitties 应用程序的基本功能也可以抽象为其他类似 NFT 的用例。
最重要的是，到这里，通过本教程您应该已经具备创建您自己pallet的逻辑和可调度函数所需知识。

## 下一步

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
