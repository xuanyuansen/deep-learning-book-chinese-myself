##第十六章：（深度学习的结构概率模型）Structured Probabilistic Models for Deep Learning 深度学习借鉴了许多建模形式，研究人员可以使用它们来指导他们的设计努力和描述他们的算法。 这些形式中的一种就是结构化概率模型的概念。 我们已经在章节3.14中简要讨论了结构化概率模型。这个简短的演示足以理解如何使用结构化概率模型作为一种语言，来描述第二部分中的一些算法。 现在，在第三部分中，结构化概率模型是深度学习中许多最重要研究课题的关键因素。 为了准备讨论这些研究思想，本章更详细地描述了结构化概率模型。本章旨在自成一体; 在继续阅读本章之前，读者不需要复习之前的介绍。
结构化概率模型是描述概率分布的一种方式，其使用图来描述概率分布中的哪些随机变量直接相互作用。这里我们使用图论理论中的“graph” ——通过一组边相互连接的一组顶点。 因为模型的结构由图形定义，所以这些模型通常也称为图形模型（graphical models）。图模型的研究社区很大，并且已经开发了许多不同的模型，训练算法和推理算法。 在本章中，我们提供了一些关于图形模型最中心思想的基本背景，并强调已经被证明对深度学习研究界最有用的概念。 如果你已经在图模型中有很强的背景，你可能希望跳过本章的大部分内容。 然而，即使图模型专家也可以从阅读本章最后一节获益，即章节16.7，其中我们强调了一些图模型用于深度学习算法的独特方式。 深度学习实习者倾向于使用非常不同的模型结构，学习算法和推理过程，而不是通常由图模型研究社区中其余人使用的。 在本章中，我们确定这些偏好的差异，并解释其原因。
在本章中，我们首先描述了建立大规模概率模型的挑战。 接下来，我们描述了如何使用图来描述概率分布的结构。 虽然这种方法使我们能够克服许多挑战，但它有自己的复杂性。 图建模中的主要困难之一是理解哪些变量需要能够直接交互，即哪些图结构最适合于给定问题。 我们通过学习章节16.5中的依赖，提出了两种方法来解决这个困难。 最后，我们在章节16.7以深度学习实践者针对图建模特别强调的具体方法的讨论结束。###16.1（非结构化建模的挑战）The Challenge of Unstructured Modeling 深度学习的目标是将机器学习扩展到解决人工智能需要面对的各种挑战。这意味着（深度学习）能够理解具有丰富结构的高维数据。例如，我们希望AI算法能够理解自然图像，表示语音的一个音频波形，以及包含多个单词和标点符号的文档。
分类算法可以从这样丰富的高维分布中获取输入，并用分类标签来描述概括，例如照片中的什么对象，记录中说出什么词，文档是什么主题。分类过程丢弃了输入中的大部分信息，并产生单个输出（或针对单个输出值的概率分布）。分类器通常也能够忽略输入的许多部分。例如，当识别照片中的对象时，通常可以忽略照片的背景。
有可能要求概率模型做许多其他任务。这些任务通常比分类开销更大。其中一些需要产生多个输出值。大多数（任务）需要完全理解输入的整个结构，没有可以选择忽略的部分。这些任务包括：
*	密度估计（Density estimation）：给定输入x，机器学习系统返回数据生成分布下的真实密度ρ（x）的估计。这只需要一个输出，但它需要完全理解整个输入。如果矢量中的一个元素是罕见的，则系统必须给它赋予低概率。
*	去噪（Denoising）：如果输入x损坏或者不正确，机器学习系统需要返回原始或正确x的估计值。例如，可能要求机器学习系统从旧照片去除灰尘或划痕。这需要多个输出（估计出的未被污染的每个元素x）和对整个输入的理解（因为即使一个损坏的区域也会是的最终的估计显示被损坏）。
*	缺失值填充（Missing value imputation）：给定x中部分元素的观察值，要求模型返回x中部分或所有未观察元素的估计或概率分布。这需要多个输出。因为模型被要求能够恢复x的任何元素，所以它必须理解整个输入。
*	采样（Sampling）：模型从分布p（x）生成新样本。这样的应用包括语音合成，即产生听起来像人类自然语音的新波形。这需要多个输出值和基于整个输入的较好模型。甚至如果样本只有一个元素来自错误的分布，那么抽样过程也是错误的。
有关在小尺寸的自然图像上进行采样任务的示例，参见图16.1。
在数千或数百万个随机变量上进行复杂分布建模是一项具有挑战性的任务，无论是计算还是统计。假设我们只想模拟二进制变量。这可能是最简单的情况，但也已经似乎是压倒性复杂的。对于小的32×32像素的颜色（RGB）图像，可能存在23072个这种形式的的二进制图像。这个数字比宇宙中估计的原子数大10800多倍。
一般来说，如果我们希望对包含k个可取值的n个离散变量的随机向量x的分布建模，则通过存储每个可能结果的一个概率值的查找表来表示P（x）的朴素方法需要 KN个参数！
这是不可行的，有如下几个原因：
*	•内存（Memory）：存储表征的成本：对于n和k的所有非常小的值，将表征分布作为表的话将需要存储太多的值。*	•统计效率（Statistical efficiency）：随着模型中参数数量的增加，使用统计估计选择这些参数的值所需训练数据的数量也增加。因为基于表的模型具有天文数量的参数，所以它将需要一个天文大的训练集来准确拟合。除非作出连接表中不同条目的附加假设（例如像在回退或平滑n-gram模型中，第12.4.1节中），任何这样的模型都将在训练集上非常严重地过度拟合。*	•运行时间（Runtime）：推理成本：假设我们要执行推理任务，其中我们使用联合分布P（x）的模型来计算一些其他分布，例如边际分布P（x1）或条件分布P （x2 | x1）。计算这些分布将需要对整个表进行求和，因此这些操作的运行时间与存储模型所需的不可控存储器成本一样高。*	•运行时间（Runtime）：抽样成本：同样，假设我们要从模型中抽取一个样本。这样做的朴素方法是抽样一些值u〜U（0,1），然后通过迭代概率表来累加概率值，直到它们超过u，并返回其最后添加的概率值结果。在最坏的情况下这需要读取整个表，因此它具有与其他操作相同的指数级别的成本。
基于表的方法的问题是我们要明确地建模每个可能的变量子集之间的每种可能的交互。我们在实际任务中遇到的概率分布比这更简单。通常，大多数变量仅间接地相互影响。
例如，考虑在接力赛中对一个团队的完成时间建模。假设团队由三个跑步者组成：爱丽丝，鲍勃和卡罗尔。在比赛开始的时候，爱丽丝拿着一根指挥棒，开始在赛道上跑。在她完成一圈赛跑后，她把指挥棒交给鲍勃。鲍勃然后自己跑，再把指挥棒交给卡罗尔，卡罗尔跑完最后一圈。我们可以将每个完成时间建模为连续的随机变量。爱丽丝的完成时间不依赖于任何人，因为她走在第一位。鲍勃的完成时间取决于爱丽丝的，因为直到爱丽丝完成她的赛跑，鲍勃才有机会开始他的那一圈。如果爱丽丝结束得更快，鲍勃会更快地完成，一切都是平等的。最后，卡罗尔的完成时间取决于她的队友。如果爱丽丝慢，鲍勃也许会结束的晚。因此，卡罗尔将的开始时间会相当晚，也很可能完成的时间较晚。然而，卡罗尔的完成时间仅通过鲍勃的完成时间而间接取决于爱丽丝的（完成时间）。如果我们已经知道鲍勃的完成时间，我们将无法通过了解爱丽丝的完成时间来更好地估计卡罗尔的完成时间。这意味着我们可以仅使用两个交互关系来模拟竞赛：爱丽丝对鲍勃的影响和鲍勃对卡罗尔的影响。我们可以从我们的模型中省略第三个，即爱丽丝和卡罗尔之间的间接交互关系。
结构化概率模型提供了一种仅用于对随机变量之间的直接相互作用进行建模的形式框架。这允许模型明显具有较少的参数，而这些参数又可以从较少的数据中可靠地估计。这些较小的模型在模型存储，在模型中执行推理以及从模型中抽样样本方面显著降低了计算成本。
###16.2 使用图来描述模型结构（Using Graphs to Describe Model Structure）结构化概率模型使用图（在由边连接的“节点”或“顶点”的图论理的论意义上）来表示随机变量之间的相互作用。 每个节点表示随机变量。 每个边表示直接交互作用。有这些节点间的直接相互作用，也意味着其他节点间的间接相互作用，但只有直接的相互作用需要被明确地建模。有多种使用图来描述概率分布中交互作用的方法。 在下面的章节中，我们描述了一些最流行和有用的方法。 图形模型可以大致分为两类：基于有向无环图的模型和基于无向图的模型。
####16.2.1	有向图模型（Directed Models）一种结构化概率模型是有向图形模型，或者称为置信网络或贝叶斯网络（Pearl，1985）。有向图形模型被称为“有向”，因为它们的边是有向的，即它们从一个顶点指向另一个顶点。该方向在图中用箭头表示。箭头的方向表示哪个变量的概率分布是根据对方来定义的。箭头从a到b意味着我们通过条件分布定义b上的概率分布，其中a作为条件栏右侧的变量之一。换句话说，b上的分布取决于a的值。
继续章节16.1中的竞赛示例。假设我们命名爱丽丝的完成时间t0，鲍勃的完成时间t1和卡罗尔的完成时间t2。正如我们前面看到的，我们对t1的估计取决于t0。我们对t2的估计直接取决于t1，但只间接取决于t0。我们可以在有向图形模型中绘制这种关系，如图16.2所示。
正式地，由变量x定义的有向图形模型由有向无环图G（其节点是模型中的随机变量）和一组局部条件概率分布p(xi | PaG(xi))定义，其中PaG（xi） 给出了G中xi的父母节点。在x上的概率分布由下式给出，
p(x)=Πp(x |Pa (x))。
在我们的比赛延迟示例中，这意味着使用图16.2中的图结构，则有，p(t0,t1,t2) = p(t0)p(t1 | t0 )p(t2 | t1)。
这是我们第一次在实践中看到一个结构化的概率模型。我们可以检查使用它的成本，以便观察结构化建模相对于非结构化建模的许多优点。
假设我们将分钟0到分钟10的时间通过6秒的时间区块来离散表示。这将使t0，t1和t2为各自具有100个可能值的离散变量。 如果我们尝试用表格表示p（t0，t1，t2），则需要存储999,999个值（100×100×100个值，再减去1，因为通过概率和为1的约束使得其成为冗余的）。相反地，如果我们只为每个条件概率分布制定一个表，那么t0上的分布需要99个值，定义给定t0时t1值的表需要9900个值，同理可得定义给定t1时t2值的表。这总共有19,899个值。 这意味着使用有向图形模型将我们的参数数量减少了超过50倍！
一般来说，为了对每个具有k个值的n个离散变量建模，单个表格方法的成本像O（kn）那样缩放，如我们之前所观察到的。 现在假设我们在这些变量上构建一个有向图形模型端条的任一侧），则有向模型的表的成本像O（km）那样缩放。 只要我们可以设计一个模型，m << n，我们得到非常显着的节省。
换句话说，只要每个变量在图中具有很少的父节点，则可以用非常少的参数来表示分布。 对图结构的一些限制，例如要求它是树，还可以保证诸如计算变量子集上的边缘或条件分布的操作是有效的。
一般来说，为了对每个具有k个值的n个离散变量建模，单个表格方法的成本类比于O（kn），如我们之前所观察到的。 现在假设我们在这些变量上构建一个有向图形模型。 如果m是在单个条件概率分布中变量出现的最大数量（在条件端的任一侧），则有向图模型的表的成本类比于O（km）。 只要我们可以设计一个模型，m << n，我们就得到非常明显的节省。
换句话说，只要每个变量在图中具有很少的父节点，则可以用非常少的参数来表示这个分布。 对图结构的一些限制，还可以保证诸如计算变量子集上的边缘或条件分布的操作是有效的，例如要求其是树。
意识到什么样的信息可以和不能在图中编码是重要的。该图仅编码在条件上彼此独立的变量的相关简化假设。还可以进行其他种类的简化假设。例如，假设我们假定鲍勃总是相同时间跑完，而不管爱丽丝何时跑完。（实际上，爱丽丝的表现可能会影响鲍勃的表现 - 根据鲍勃的个性，如果爱丽丝在一场比赛中跑得特别快，这可能激励鲍勃努力来与她的特殊表现相匹配，或者可能会让他过分自信和懒惰）。然后，爱丽丝对鲍勃的完成时间的唯一影响是，我们必须将爱丽丝的完成时间加到我们认为鲍勃需要完成的总时间。这个观察允许我们定义有O（k）参数的模型，而不是O（k2）。然而，请注意t0和t1仍然直接依赖于这个假设，因为t1表示鲍勃完成的绝对时间，而不是他自己花费的总时间。这意味着我们的图形必须包含从t0到t1的箭头。鲍勃的个人完成时间独立于所有其他因素的假设，所以不能在t0，t1和t2的图中编码。相反，我们在条件分布本身的定义中对此信息进行编码。条件分布不再是由t0和t 1索引的k×k-1的元素表，但现在是仅使用k-1个参数的略微更复杂的公式。有向图形模型语法不对我们如何定义条件分布施加任何约束，它只定义了允许哪些变量被作为参数接受。
####16.2.2	无向图模型（Undirected Models）有向图形模型给了我们一种描述结构化概率模型的语言。另一种流行的语言是无向模型，也称为马尔科夫随机场（MRFs）或马尔可夫网络（Kindermann，1980）。顾名思义，无向模型使用的图的边是没有方向的。
有向模型最适用于有明确理由在每个特定方向上绘制一个箭头的情况。通常这些是我们理解的因果关系，且这种情况下因果关系只有一个方向。一种这样的情况是接力跑赛的示例。先前的跑步者影响之后跑步者的完成时间；之后跑步者不影响先前跑步者的完成时间。
不是所有的情况下我们都想要建模这样一个明确的方向和他们之间的交互作用。当这个交互似乎没有内在方向或者在两个方向上都存在时，使用无向图模型可能更合适。
作为这种情况的一个例子是，假设我们要对三个二进制变量建模：无论你是否生病，你的同事是否生病，以及你的室友是否生病。如在接力赛例子中，我们可以对发生的交互作用的种类做出简化的假设。假设你的同事和你的室友彼此不认识，他们中的一个人不可能直接传染给另一个人疾病，如感冒。这个事件可以看作是非常罕见的，以至于它不能被建模。然而，很可能他们中的任何一个可以传染给你感冒，你可以把它传递给另一个人。我们可以通过建模从你的同事到你的感冒传染和从你到你的室友的感冒传染，模拟从你的同事到你的室友间感冒的间接传输。
在这种情况下，你很容易让你的室友生病，同时你的室友也很容易让你生病，所以没有一个清晰的单向的叙事并以此为基础建立模型。这促使了无向图模型的使用。与有向图模型一样，如果无向图模型中的两个节点通过边连接，则与那些节点相对应的随机变量直接相互交互。与有向图模型不同的是，无向图模型中的边没有箭头，并且不与条件概率分布相关联。
我们将表示你的健康程度的随机变量表示为hy，表示室友的健康程度的随机变量表示为hr，表示你的同事的健康程度的随机变量为hc。参用于表示这种情况的图如16.3所示。
形式上，无向图模型是在无向图G上定义的结构化概率模型。对于图中的每个团C，因子φ（C）（也称为团的势能）测量变量处于每个可能的联合状态时在该团中的亲和程度。这些因子被约束为非负的。它们一起定义了非规范化的概率分布，
p ̃(x) = ΠC∈G φ(C)
只要所有的团都很小，非规范化的概率分布对于工作是有效的。它编码了这样的想法，即具有更高亲和力的状态是更有可能的。 然而，不像在贝叶斯网络中，团的定义中几乎没有结构信息，因此没有什么可以保证将它们相乘在一起将产生有效的概率分布。图16.4展示了从无向图读取因子分解信息的示例。
你，你的室友和你的同事间感冒传播的例子包含两个团。一个团包含hy和hc。这个团的因子可以由表定义，并且可能具有类似于这些值：状态1表示健康良好，而状态0表示健康不良（即已经感冒）。你们两个都通常是健康的，所以相应的状态具有最高的亲和力。只有一个人生病的状态具有最低的亲和力，因为这是一个罕见的状态。你们两个人都病了（因为你们中有一个感染了另一个）的状态是一个更高的亲和状态，虽然仍不如两个人都健康的状态常见。
为了完成模型，我们还需要定义一个类似的因子，包含hy和hr的团。
####16.2.3	配分函数（The Partition Function）虽然非规范化的概率分布保证在任何地方都是非负的，但是不能保证其求和或积分为1。为了获得有效的概率分布，我们必须使用相应的归一化概率的分布，
p(x) = Z1 p ̃(x) 
其中Z使得概率分布求和或者的值积分为1。
当φ函数保持不变时，您可以将Z看作常数。注意，如果φ函数具有参数，则Z是这些参数的函数。在文献中常见的是省略Z的参数来节省空间。
归一化常数Z称为配分函数，一个借用了统计物理学的术语。
由于Z是状态x的所有可能值的联合积分或者求和，所以计算通常是棘手的。为了能够获得无向图模型的归一化概率分布，模型结构和φ函数的定义必须有利于有效地计算Z. 在深度学习的背景下，Z的计算通常是棘手的。 由于难以计算Z，我们必须求助于近似方法。这种近似算法是第18章的主题。
在设计无向图模型时要记住的一个重要考虑因素是，可以在Z不存在的情况下指定因子。如果模型中的某些变量是连续的，并且p在它们域上的积分发散，就会发生这种情况。例如，假设我们想要用单个团的势能φ（x）= x2来建模单个标量变量x∈R。在这种情况下，Z= x2dx. 
由于该积分发散，因此没有对应于φ（x）这种选择的概率分布。有时，φ函数的一些参数的选择决定了概率分布的定义。例如，对于φ（x;β）= exp(-βx2)，β参数决定了Z是否存在。正β产生在x上的高斯分布，但是所有其他的β值都使得φ不可能归一化。
有向图模型和无向图模型之间的一个主要区别是有向图模型直接从概率分布开始定义，而无向图模型则通过φ函数来更宽松地定义，然后再转换为概率分布。这改变了为了使用这些模型而必须具有的有关模型的直觉。在使用无向图模型时要记住的一个关键点是，每个变量的域对给定的一组φ函数对应概率分布的种类有显著的影响。例如，考虑n维随机变量的向量x和由偏置向量b参数化的无向图模型。假设我们对x的每个元素有一个团体，φ(xi)= exp(bixi)。这会导致什么样的概率分布？答案是我们没有足够的信息，因为我们还没有指定x的域。如果x∈Rn，则通过积分定义的Z会发散并且不存在概率分布。如果x∈{0,1} n，则p（x）分解为n个独立分布，其中p（x i = 1）= sigmoid（bi）。如果x的域是基本基向量（{[1,0，...，0]，[0,1，...，0]，...，[0,0，...] ，1}）），则p（x）= softmax（b），因此对于j!= i，bi的较大值实际上减小了p（x j = 1）。通常，可以利用仔细选择变量域的效果，以便从相对简单的一组φ函数中获得复杂的行为。我们将在后面的章节20.6中探讨这个想法的实际应用。
####16.2.4	基于能量的模型（Energy-Based Models）许多关于无向图模型的有趣的理论结果取决于∀x，p（x）> 0的假设。强制确保该条件的一种简便方法是使用基于能量的模型（EBM），其中，
p ̃(x) = exp(−E(x))
而E（x）被称为能量函数。因为exp（z）对所有z都是正的，这保证了能量函数在任何状态x下的概率都不为零。
完全自由地选择能量函数使学习变得更简单。如果我们想直接学习团的势能，我们需要使用约束优化任意强加到一些特定的最小概率值。 通过学习能量函数，我们可以使用无约束优化。在基于能量的模型中概率可以任意接近零，但是永远不会为零。图16.4 
任何由方程式16.7给出的分布形式都是波尔兹曼分布的示例。为此，许多基于能量的模型被称为波尔兹曼机（Fahlman et al., 1983; Ackley et al., 1985; Hinton et al., 1984; Hinton and Sejnowski, 1986）。什么时候称模型为基于能量的模型，什么时候称其为玻尔兹曼机，并没有一个广为接受的准则。Boltzmann机首先引入时是来描述一个只有二进制变量的模型，但是今天的许多模型也结合了实值变量，如平均方差-协方差限制的Boltzmann机。虽然玻尔兹曼机最初被定义包括了具有和不具有隐变量的模型，但是今天术语波尔兹曼机最常用于指定具有隐变量的模型，而没有隐变量的玻尔兹曼机更常被称为马尔可夫随机场或对数线性模型。图16.5
无向图中的团对应于非规范化的概率函数因子。因为exp（a）exp（b）= exp（a + b），这意味着无向图中的不同团对应于能量函数的不同项。换句话说，基于能量的模型只是一种特殊的马尔可夫网络：求幂使得能量函数中的每个项对应于不同团的因子。参见图16.5中的示例，如何从无向图结构中读取能量函数的形式。人们可以把基于能量的模型中能量函数的每一项看是专家的乘积（Hinton，1999）。能量函数中的每项对应于概率分布中的一个因子。能量函数的每个项可以被认为是确定是否满足特定软约束的“专家”。每个专家可以仅执行仅涉及随机变量的低维投影的一个约束，但是当通过概率的乘法组合时，专家一起强制执行复杂的高维约束。
从机器学习观点来看，基于能量的模型的定义中的一部分不起功能作用：如等式16.7中的符号“-”。这个符号可以被合并到E的定义中，或者对于许多函数E，学习算法可以简单地学习具有相反符号的参数。符号的存在主要是为了保持机器学习文献和物理学文献之间的兼容性。许多概率建模的进展最初由统计物理学家开发，其中E是指实际的，物理的能量，没有任意的符号。诸如“能量”和“分区函数”这些术语仍然与这些技术相关联，即使它们的数学适用性比它们被发明的物理学上下文含义更宽。 一些机器学习研究者（例如，Smolensky（1986）将负能量称为和谐）选择忽略对立面，但这并不是标准惯例。
许多算法对概率模型进行操作时并不需要计算pmodel（x），而只需要计算log p model（x）。 对于包含隐变量h的基于能量的模型，这些算法有时根据该量的负数来表示，称为自由能：
在这本书中，我们通常倾向更一般的log pmodel（x）公式。
####16.2.5	分离和D分离（Separation and D-Separation）图模型中的边告诉我们哪些变量直接交互。我们也经常需要知道哪些变量有间接相互作用。某些间接交互可以通过观察其他变量来启用或禁用。更正式地，在给定其他变量子集的值的情况下，我们想知道哪些变量子集在条件上彼此独立。
在无向图模型的情况下，在图中识别条件独立性是非常简单的。在这种情况下，图中隐含的条件独立性称为分离。如果在给定S的情况下，图结构暗示A与B无关，那么我们说给定第三组变量S，将一组变量A与另一组变量B分离。如果两个变量a和b通过仅涉及未观察到的变量，那么这些变量不分离。如果它们之间没有路径，或者所有路径都包含一个观察变量，那么它们是分开的。我们将仅涉及未观察变量的路径称为“活动”，将包括观察到的变量的路径称为“非活动”。图16.6：（a）随机变量a和随机变量b到s之间的路径是有效的，因为没有观察到s。 这意味着a和b不分离。（b）这里的s用阴影表示，表示被观察到。 因为a和b之间的唯一路径是通过s的，并且该路径是非活动的，所以我们可以得出结论，a和b是由给定的s分离的。
当我们画一个图，我们可以通过阴影来表示观察到的变量。图16.6用于描述当以这种方式绘制时，无向图模型中活动和非活动路径的样子。 参见图16.7为从无向图读取分离的例子。图16.7：从无向图中读取分离属性的示例。 这里用阴影表示b被观察到。 因为观察b阻挡了从a到c的唯一路径，所以我们说a和c在给定b的情况下彼此分离。b的观察也阻塞了a和d之间的一个路径，但是它们之间存在第二个活动路径。因此，a和d在给定b时不分离。
类似的概念适用于有向图模型，除了在有向图模型的上下文中，这些概念被称为d分离。“d”代表“依赖性”。有向图中D分离的定义与无向图中的分离相同：我们说，给定第三组变量S，如果A独立于给定的B，那么这样的图结构意味着一组变量A与另一组变量B是d分离的。图16.8：长度为2的所有类型的活动路径，它们可以存在于随机变量a和b之间。（a）具有直接从a到b或反之亦然的箭头的任何路径。如果观察到s，这种路径被阻塞。我们已经在接力竞赛示例中看到了这种路径。（b）a和b通过共同原因连接。例如，假设s是指示是否存在飓风的变量，并且a和b测量在两个不同的附近天气监视前哨处的风速。如果我们在站a观察到非常强的风，我们可能期望在b处看到强风。这种路径可以通过观察s来阻止。如果我们已经知道有飓风，我们期望在b处看到大风，而不管在a处观察到什么。低于预期的风（在飓风）不会改变我们对风的预期b（知道有飓风）。然而，如果没有观察到s，则a和b是相关的，即路径是活动的。（c）a和b都是s的父母。这称为V形结构或碰撞体。V结构通过解释离散效应使a和b相关。在这种情况下，当观察到s时，路径实际上是活动的。例如，假设s是一个表示您的同事不在工作的变量。变量a表示她正在生病，而b表示她正在度假。如果你观察到她不在工作，你可以推测她可能病了或在度假，但是不是特别可能两个发生在同一时间。如果你发现她在度假，这个事实就足以解释她的缺席。你可以推断她可能没有生病。（d）这解释了分离效应的发生，即使观察到任何s的后代！例如，假设c是一个变量，表示您是否收到了同事的报告。如果你注意到你没有收到报告，这增加了她今天不在工作的概率的估计，这反过来又使她很可能生病或度假。通过V结构阻塞路径的唯一方法是观察共享子节点的后代。
与无向模型一样，我们可以通过查看图中存在的活动路径来检查图中隐含的独立性。 如前所述，如果两个变量之间存在活动路径，则两个变量是相关的，如果不存在这样的路径，则为d分离的。 在有向网络中，确定路径是否活动有点更复杂。在有向图模型中识别活动路径的示例参见图16.8。从一个图中读取一些属性的示例参见图16.9。
图16.9：从该图中，我们可以读出几个d分离特性。示例包括：*	•a和b在给定空集时是d分离的。*	•a和e在给定c时是d分离的。*	•d和e在给定c时是d分离的。
我们还可以看到，当我们观察一些变量时，一些变量不再是d分离的：
*	•a和b在给定c时不是d分离的。 *	•a和b在给定d时不是d分离的。
重要的是要记住，分离和D分离只告诉我们图中隐含的条件独立性。不需要该图表示所有存在的独立性。特别地，使用完整图（具有所有可能的边的图）来表示任何分布总是合法的。事实上，一些分布包含不可能用现有图形符号表示的独立性。上下文特定的独立性是依赖于网络中一些变量的值而存在的独立性。例如，考虑一个三个二进制变量的模型：a，b和c。假设当a是0时，b和c是独立的，但是当a是1时，b确定地等于c。当a = 1时对行为进行编码需要连接b和c的边。那么该图就不能表示当a = 0时b和c是独立的。
一般来说，图永远不会暗示不存在的独立性。然而，图很可能无法编码一个独立性。
####16.2.6	有向图与无向图之间的转化（Converting between Undirected and Directed Graphs）我们经常将特定的机器学习模型称为无向的或有向的。 例如，我们通常将RBM称为无向的，将稀疏编码器称有向的。 这种措辞的选择可能有点误导，因为没有概率模型是固定有向的或无向的。相反，一些模型使用有向图来描述最容易，或使用无向图来描述最容易。有向模型和无向模型都有其优点和缺点。没有哪一种方法是明显优越和普遍优选的。相反，我们应该选择每个任务使用哪种语言。这个选择将部分取决于我们希望描述的概率分布。我们可以基于哪种方法可以捕获概率分布中的最显著的独立性，或者哪种方法可以使用最少的边来描述分布，来选择使用有向建模或无向建模。还有其他因素可以影响决定使用哪种语言。即使在使用单个概率分布时，我们有时也可以在不同的建模语言之间切换。有时，如果我们观察到变量的某个子集，或者如果我们希望执行不同的计算任务，则不同的语言变得更合适。例如，有向模型描述通常提供一种直接的方法来有效地从模型中抽取样本（在第16.3节中描述），而无向模型公式通常用于推导近似推理过程（我们将在第19章中看到，无向模型的作用在等式19.56中会突显）。每个概率分布都可以由有向模型或无向模型表示。 在最坏的情况下，可以通过使用“完整图”来表示任何分布。在有向模型的情况下，完整图可以是任意的有向无环图，其中我们对随机变量施加一些排序约束，并且每个变量具有所有在排序中位于其前面的其他变量作为其在图中的祖先。对于无向模型，完整图只是一个包含所有变量的单个团（clique）的图形。参见图16.10中的示例。图16.10：可以描述任何概率分布的完整图的示例。这里我们展示了四个随机变量的例子。（左）完整的无向图。在无向图的情况下，完整图是唯一的。（右）完整的有向图。在有向的情况下，没有唯一的完整图。我们按顺序选择变量，并从每个变量绘制一个弧到在其排序后的每个变量。 因此，对于每组随机变量存在因子个数的完整图。 在这个例子中，我们从左到右，从上到下排序变量。
当然，图模型的效用是图意味着一些变量不直接交互。 完整的图不是非常有用，因为它不暗示任何独立性。
当我们用图表示概率分布时，我们想要选择一个图，其意味着尽可能多的独立性，而不暗示任何实际上不存在的独立性。
从这个角度来看，一些分布可以使用有向图模型更有效地表示，而其他分布可以使用无向图模型更有效地表示。 换句话说，有向图模型可以编码一些独立性，而无向图模型不能编码，反之亦然。
有向模型能够使用一种特定类型的子结构，无向模型不能完美地表示。 这个子结构被称为不道德。当两个随机变量a和b都是第三个随机变量c的父亲，并且在任一方向上没有边直接连接a和b时，该结构发生。（“不道德”的名称可能看起来很奇怪;它在图模型文献中是作为一个关于未婚父母的笑话创造的）。要将带有图D的有向图模型转换为无向图模型，我们需要创建一个新的图U。对于每个变量对x和y，如果存在将D中的x和y连接的有向边（在任一方向上）或者如果x和y都是第三变量z在D中的父节点，则在U中添加将x和y连接的无向边。生成的U被称为道德化的图。参见图16.11中关于通过道德化将有向模型转化为无向模型的例子。
同样，无向图模型可以包括有向图模型不能完美表示的子结构。具体来说，如果U包含长度大于3的循环，则有向图D不能捕获无向图U所暗示的所有条件独立性，除非该循环还包含和弦（chord）。循环是由无向边连接的变量序列，序列中的最后一个变量连接回序列中的第一个变量。和弦（chord）是定义循环的序列中任意两个非连续变量之间的连接。如果U具有长度为4或更大的循环，并且没有这些循环的和弦，我们必须在将它们转换为有向图模型之前添加和弦。添加这些和弦丢弃了在U中编码的一些独立行信息。通过将和弦添加到U形成的图被称为弦或三角形图，因为现在可以用更小的三角形环来描述所有的环。要从弦图构建有向图D，我们还需要为边指定方向。当这样做时，我们不能在D中创建有向循环，否则结果就是不能定义有效的有向概率图模型。为D中的边分配方向的一种方法是对随机变量施加排序，然后将每个边从排序较早的节点指向排序靠后的节点。参见图16.12中的演示。
####16.2.7	因子图（Factor Graphs）因子图是绘制无向图模型的另一种方式，用来解决在使用标准无向图模型语法的图形中表示模糊性。在无向图模型中，每个φ函数的范围必须是图中某个团的子集。然而，不必存在任何φ其范围包含每个团体的整体。因子图明确表示每个φ函数的范围。具体来说，因子图是由二分无向图组成的无向模型的图形表示。一些节点被绘制为圆形。这些节点对应于随机变量，如在标准无向图模型中。其余节点绘制为正方形。这些节点对应于非归一化概率分布的因子φ。变量和因子可以与无向边连接。当且仅当变量是非归一化概率分布中的因子的参数之一时，变量和因子在图中连接。没有因子可以连接到图中的另一个因子，也不能将变量连接到变量。参见图16.13中的示例，说明了因子图如何解决无向网络解释中的模糊性。
###16.3	从图模型中抽样（Sampling from Graphical Models）图模型还有利于从模型中抽取样本的任务。有向图模型的一个优点是，一个简单且有效的称为祖先采样的过程可以从由模型表示的联合分布中产生样本。
基本思想是将图中的变量xi排序为拓扑排序，使得对于所有i和j，如果xi是xj的父代，则j大于i。然后可以按此顺序对变量进行采样。换句话说，我们首先采样x1〜P（x1），然后采样P（x2 | PaG（x2）），等等，直到最后我们采样P（xn | PaG（xn））。只要每个条件分布p（xi | P aG（xi））易于从中抽样，则整个模型容易从中抽样。拓扑排序操作保证了我们可以按顺序读取方程16.1中的条件分布和它们的样本。如果没有拓扑排序，我们可能会尝试在父节点可用之前对变量进行抽样。
对于一些图，可能有多个拓扑排序。祖先采样可以与这些拓扑排序中的任何一个一起使用。祖先采样通常非常快（假设从每个条件的采样容易）并且方便。
祖先采样的一个缺点是其仅适用于有向图模型。另一个缺点是它不支持每个条件采样操作。给定一些其他变量，当我们希望从有向图模型中的变量子集中抽样时，我们经常要求所有的条件变量比有序图中要抽样的变量更早。在这种情况下，我们可以从模型分布指定的局部条件概率分布进行抽样。否则，我们需要采样的条件分布是给定观测变量的后验分布。这些后验分布通常没有明确指定并在模型中参数化。 推断这些后验分布可能是昂贵的。在这种情况下的模型中，祖先采样不再有效。
不幸的是，祖先采样仅适用于有向图模型。我们可以通过将它们转换为有向图模型来从无向图模型中抽样，但是这通常需要解决棘手的推理问题（以确定新有向图的根节点上的边缘分布）或者需要引入这么多边，使得得到的有向图模型变得棘手。从无向图模型抽样，而不首先将其转换为有向图模型似乎需要解决循环依赖。每个变量与其他每个变量交互，因此抽样过程没有明确的起点。不幸的是，从无向图形模型绘制样本是一个昂贵的多通道过程。概念上最简单的方法是吉布斯抽样（Gibbs sampling）。假设我们在随机变量x的n维向量上有一个图模型。我们迭代地访问每个变量xi，并从p（x​​i | x-i）绘制一个条件为所有其他变量的样本。由于图模型的分离性质，我们可以仅对xi的邻居进行等价条件化。不幸的是，在我们通过图模型并对所有n个变量进行抽样之后，我们仍然没有来自p（x）的公平样本。相反，我们必须重复该过程并使用它们邻居的更新值对所有n个变量重新取样。渐近地，在多次重复之后，该过程收敛到从正确分布的采样。可能难以确定样本何时达到期望分布的足够精确的近似。无向图模型的抽样技术是一个高级主题，第17章将对此进行更详细的讨论。
###16.4	结构化建模的优势（Advantages of Structured Modeling）使用结构化概率模型的主要优点是它们允许我们显著降低表示概率分布以及学习和推断的成本。在有向图模型的情况下，采样也加速，而对于无向图模型，情况可能更复杂。允许所有这些操作使用较少的运行时和内存的主要机制是选择不对某些交互进行建模。图模型通过留下边来传达信息。在任何没有边的地方，模型指定了我们不需要建模直接交互的假设。
使用结构化概率模型的较少量化的益处是，它们允许我们明确地将知识的表示与知识的学习或给定现有知识进行推断分开。这使我们的模型更容易开发和调试。我们可以设计，分析和评估适用于大多数图的学习算法和推理算法。独立地，我们可以设计捕捉我们认为在我们的数据中的重要关系的模型。然后，我们可以组合这些不同的算法和结构，并获得不同可能性的笛卡尔乘积。为每种可能的情况设计端到端算法将更加困难。
###16.5	学习依赖关系（Learning about Dependencies）良好的生成模型需要准确地捕获所观察到“可见”变量v上的分布。通常v的不同元素高度依赖于彼此。在深度学习的上下文中，最常用于建模这些依赖关系的方法是引入几个潜在的或“隐藏”变量h。然后，该模型可以通过vi和h之间的直接依赖性以及h和vj之间的直接依赖性间接地捕获任何一对变量vi和vj之间的依赖性。
不包含任何隐变量的v的良好模型将需要在贝叶斯网络中的每个节点具有非常大数量的父节点或在马尔可夫网络中具有非常大的团。仅仅代表这些高阶交互是昂贵的——无论是在计算意义上，因为必须存储在存储器中的参数的数量与团体中的成员的数量成指数地缩放，还是在统计学意义上，因为这个指数数量的参数需要大量的数据来准确估计。
当模型旨在捕获具有直接连接的可见变量之间的依赖关系时，通常不可能连接所有变量，因此图必须设计为连接紧密耦合的那些变量并省略其他变量之间的边。机器学习的一整个领域称为结构学习专门讨论这个问题。关于结构学习的一个好的参考，见（Koller和Friedman，2009）。大多数结构学习技术是一种贪婪搜索的形式。提出了一种结构，对具有该结构的模型进行训练，然后给出分数。该分数奖励高训练集精度并惩罚模型复杂性。然后提出添加或移除少量边的候选结构作为搜索的下一步。搜索进行到预期增加得分的新结构。
使用隐变量而不是自适应结构避免了执行离散搜索和多轮训练的需要。可见变量和隐变量之间的固定结构可以使用可见单元和隐单元之间的直接交互，以强制可见单元之间的间接交互。使用简单的参数学习技术，我们可以学习一个具有固定结构的模型，在边界p（v）上输入正确的结构。隐变量具有超越它们有效捕获p（v）的作用的优点。新变量h还为v提供了另一种表示形式。例如3.9.6中的讨论，高斯混合模型学习了一个隐变量，它对应于从哪个类别的实例中抽取输入。这意味着高斯混合模型中的隐变量可以用于做分类。在第14章中，我们看到了简单的概率模型如稀疏编码学习隐变量，可以用作分类器的输入特征或沿着歧管的坐标。其他模型可以以相同的方式使用，但是具有不同种类交互的更深的模型和模型可以创建甚至更丰富的输入描述。许多方法通过学习隐变量来完成特征学习。通常，给定v和h的一些模型，实验观察显示E [h | v]或argmaxhp（h，v）是v的良好特征映射。
###16.6	推理和概率推理（Inference and Approximate Inference）我们使用概率模型的主要方法之一是，提出关于变量如何相互关联的问题。给定一组医学测试，我们可以询问患者可能患有什么疾病。在隐变量模型中，我们可能需要提取特征E [h | v]描述观察变量v。有时我们需要解决这些问题，以执行其他任务。我们经常使用最大似然原则来训练我们的模型。因为
log p(v) = Eh∼p(h|v) [log p(h, v) − log p(h | v)]
我们经常想要计算p（h | v）以实现规则学习。所有这些都是推理问题的例子，其中给定其他变量我们必须预测一些变量的值，或者在给定其他变量的值的情况下预测一些变量的概率分布。不幸的是，对于大多数有趣的深度模型，这些推理问题是棘手的，即使我们使用结构化的图模型来简化它们。图形结构允许我们用合理数量的参数来表示复杂的高维分布，但是用于深度学习的图通常不足够限制以允许有效的推断。
可以直接看出，计算一般图模型的边际概率是#P hard。复杂性类别#P hard是复杂性类别NP的泛化。NP中的问题需要仅确定问题是否有解决方案，并找到解决方案（如果存在）。#P hard中的问题需要计算解决方案的数量。为了构建最坏情况的图模型，假设我们在3-SAT问题中对二进制变量定义一个图模型。我们可以对这些变量施加均匀分布。然后我们可以为每个子句添加一个二进制隐变量，指示每个子句是否得到满足。
然后，我们可以添加另一个隐变量，指示是否满足所有子句。这可以不通过构建隐变量的缩减树而做出大的团体，其中树中的每个节点报告是否满足两个其他变量。该树的叶是每个子句的变量。树的根报告整个问题是否满足。由于文字上的均匀分布，缩小树根的边缘分布规定了什么样的分配部分满足问题。虽然这是一个设计的最坏情况的例子，NP硬图通常出现在实际现实世界的场景。
这促使了使用近似推理。在深度学习的上下文中，这通常涉及变分推理，其中通过寻求尽可能接近真实分布的近似分布q（h | v）来逼近真实分布p（h | v）。这个和其他技术将在第19章中有深入的描述。
###16.7	 结构化概率模型的深度学习方法（The Deep Learning Approach to Structured Prob- abilistic Models）深度学习实践者通常与其他机器学习实践者使用相同的结构化概率建模的基本计算工具。然而，在深度学习的上下文中，我们通常对如何组合这些工具做出不同的决定，导致总体算法和模型与传统的图模型具有非常不同的风格。
深度学习并不总是涉及特别深的图模型。在图模型的上下文中，我们可以根据图模型图而不是计算图来定义模型的深度。如果从hi到观察变量的最短路径是j步，则我们可以认为隐变量hi处于深度j。我们通常将模型的深度描述为任何这样的hi的最大深度。这种深度不同于由计算图归纳的深度。用于深度学习的许多生成模型没有隐变量或只有一层隐变量，但是可以使用深度计算图来定义模型中的条件分布。
深度学习基本上总是利用分布式表征的想法。即使是用于深度学习目的浅层模型（例如预训练浅层模型，稍后形成深层模型）也几乎总是具有单个大的隐变量层。深度学习模型通常具有比观察到的变量更多的隐变量。复杂的变量之间的非线性交互通过流经多个隐变量的间接连接来实现。相比之下，传统的图模型通常包含大部分偶尔观察到的变量，即使许多变量在一些训练样本中会随机丢失。传统模型大多使用高阶项和结构学习来捕获变量之间复杂的非线性相互作用。如果存在隐变量，那么它们的数量通常很少。
深度学习中隐变量的设计方式也不同。深度学习实践者通常不希望提前为隐变量采用任何特定语义——训练算法可以自由地发明概念来建模特定的数据集。通常对于人们来说在事后解释隐变量不是很容易，但是可视化技术可以允许它们表示一些粗略的表征。当在传统图模型的上下文中使用隐变量时，它们通常被设计成具有一些特定语义——文档的主题，学生的智力，导致患者症状的疾病等。这些模型通常是更能由人类从业者解释的，并且通常具有更多的理论保证，但是不能扩展到复杂的问题，并且不能同深层模型一样在不同背景中重复使用。
另一个明显的区别是在深度学习方法中经常使用的连接类型。深度图模型通常具有大的单元组，其都连接到其他单元组，使得两个组之间的交互可以由单个矩阵描述。传统的图模型具有非常少的连接，并且每个变量间连接的选择可以单独设计。模型结构的设计与推理算法的选择紧密相关。图模型的传统方法通常旨在保持精确推断的可追踪性。当这种约束太限制时，流行的近似推理算法是称为循环置信传播的算法（loopy belief propagation）。这两种方法通常在非常稀疏的连接图上工作得很好。通过比较，在深度学习中使用的模型倾向于将每个可见单元vi连接到非常多的隐单元hj，使得h可以提供vi的分布式表示（以及可能还有几个其他观察变量）。分布式表征具有许多优点，但是从图模型和计算复杂性的观点来看，分布式表征通常具有产生不够稀疏的图的缺点，不足以用于精确推断和循环信任传播的传统技术。因此，较大的图模型社区和深层图模型社区之间最明显的区别之一是，循环信仰传播几乎从未用于深度学习。多数深度学习模型被设计成使吉布斯抽样或变分推理算法高效。另一个考虑是深度学习模型包含非常大量的隐变量，使得有效的数字表征至关重要。除了选择高级推理算法之外，这提供了另外的动机，用于将单元分组成具有描述两个层之间相互作用的矩阵的层。这允许利用有效的矩阵乘积运算或广义稀疏连接（例如块对角矩阵乘积或卷积）来实现算法的各个步骤。
最后，图建模的深度学习方法的一个显著特征是对未知的包容性。我们增加模型的力量，直到它只几乎不可能训练或使用，直到我们可能想要的所有数量都可以精确计算，而不是简化模型。 我们经常使用边缘分布不能计算的模型，并且只是简单地从这些模型中抽样近似样本。 我们经常训练具有难以处理的目标函数的模型，我们甚至不能在合理的时间内近似，但是如果我们能够有效地获得类似函数的梯度估计，我们仍然能够近似地训练模型。深度学习方法通常是找出我们绝对需要的最小量的信息，然后找出如何尽快得到该信息的合理近似。
####16.7.1	示例：受限玻尔兹曼机（Example: The Restricted Boltzmann Machine）受限玻尔兹曼机（RBM）（Smolensky，1986）或者harmonium是图模型用于深度学习的典型例子。RBM本身并不是一个深层模型。相反，它只有一层隐变量，可用于学习输入的表示。在第20章中，我们将看到RBM如何被用来构建更多的深层模型。在这里，我们展示了RBM如何例证了深度图模型中使用的许多实践：它的单位被组织为称为层的大组，层之间的连接由矩阵描述，连接相对密集，模型被设计为允许有效的吉布斯采样，并且模型设计的重点在于释放训练算法以学习其未由设计者指定语义的隐变量。 之后我们将在章节20.2再次更详细地讨论RBM。规范的RBM是具有二进制可见和隐单元的基于能量的模型。其能量函数为，其中b，c和W是不受约束的，实值的，可学习的参数。我们可以看到，模型被分成两组单元：v和h，它们之间的相互作用由矩阵W描述。该模型如图16.14。如该图所示，该模型的一个重要方面是在任何两个可见单元之间或任何两个隐藏单元之间没有直接相互作用（因此称为“受限”，一般的玻尔兹曼机器可以具有任意连接）。
RBM结构上的限制导致了良好的限制，
p(h | v) = Πip(hi | v)
和
p(v | h) = Πip(vi | h)
单个条件也很容易计算。对于二元RBM我们有：这些属性在一起允许有效的块吉布斯采样，其在同时采样所有h和同时采样所有v之间交替。由RBM模型的Gibbs采样产生的样品如图16.15所示。
由于能量函数本身只是参数的线性函数，因此很容易得到能量函数的导数。例如，这两个属性——高效的吉布斯抽样和高效的导数——使训练方便。在第18章中，我们将看到，可以通过计算应用于来自模型的样本的这种导数来训练无向图模型。
训练模型将产生数据v的表示h。我们经常可以使用Eh〜p（h | v）[h]作为一组特征来描述v。总的来说，RBM演示了典型的图模型深度学习方法：通过隐变量层完成表示学习，结合由矩阵参数化的层之间的有效交互。
图模型的语言为描述概率模型提供了一种优雅，灵活和清晰的语言。 在前面的章节中，我们使用这种语言，以及其他视角来描述各种各样的深度概率模型。