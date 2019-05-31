# 插件开发指南  

本文介绍如何开发供Barbican使用的自定义插件。尽管Barbican提供了有用的插件实现，但有些OpenStack运营商可能需要一些定制化的实现，可能是与现有的企业数据库或服务进行交互。这种方法也为OpenStack云运营商提供了灵活性，允许他们为自己的云选择合适的实施方案。  

# 插件状态  

Barbican插件的可选状态有三种：**stable（稳定版）**、**experimental（试验性质的）** 和 **out-of-tree（不再维护）**。  
* stable状态表明该插件完成受OpenStack Barbican团队支持。  
* experimental表明我们打算支持此插件，但是它可能确实某些功能或无法进行完成的边界测试。此状态的插件可能会中断开发。  
* out-of-tree表明此插件不再提供支持，在下个版本可能会被移除。  

# 结束过程  

默认情况下，in-tree下的插件将处于试验状态。要改为stable，插件必须满足以下条件：  
* 100%的单元测试覆盖，包括分支覆盖。  
* 针对配置为使用该插件的Barbican实例执行功能测试套件中的Gate作业，可以是开发Gate，也可以是第三方Gate。  
* 在新蓝图功能被批准后，在下一个周期内实现新功能。  

# 降级过程  

插件不应该长时间处于试验状态。对于两个以上版本处于试验状态的插件，会执行结束流程，进入稳定状态，或者直接移除。  

处于稳定状态的插件可能会被团队启用并移除。  

可从开发树中删除树外状态超过两个版本以上的插件。  

# 架构  

Barbican的插件架构允许开发人员创建自己的功能实现，例如机密的存储和生成以及事件处理。使用的插件模式定义了一个抽象类，其方法由Barbican logic(在本指南中称为Barbican“core”)按特定的顺序调用。通常，插件不会直接与Barbican的数据模型交互，因此Barbican核心也会处理代表插件的任何所需信息。  

通常，Barbican core将调用插件的supports()方法的变体来确定请求的操作是否可以由插件实现。一旦选择了支持插件，Barbican core将调用插件上的一个或多个方法来完成操作。  

下面提供了有关Barbican使用的各种插件类型以及配置和部署选项的进一步指导。  

# 机密存储开发  

以下文档说明了如何开发一个供Barbican使用的定制化机密存储插件。  

Barbican提供两种机密存储的模式：机密存储模式（本章节详细介绍的）和加密模式。机密存储模式在插件中执行加解密和加密的机密存储操作。Barbican插件中包括红帽的Dogtag服务和密钥管理互操作性协议(KMIP)安全设备的插件实现。  


由于秘密存储模式将加密秘密的存储推迟到插件，所以与加密模式不同，Barbican core不需要将加密的秘密存储到其数据存储中。为了适应两种秘密存储模式之间的差异，Barbican core中包含了一个加密插件适配器的秘密存储，详见下面的加密插件适配器一节。

## secret_store 模型  

**barbican.plugin.interface.secret_store**中包含了定制化插件需要实现的所有类。这些类中包括了 **SecretStoreBase**，自定义插件应该继承的抽象类，也包括了几个用来在Barbican和插件之间传递数据的数据传输对象类（Data Transfer Object，DTO）类。  

## Data Transfer Objects  

DTO类用来打包从Barbican传到插件的数据，也返回从插件到Barbican的数据。它们在插件和Barbican的内部数据模型之间提供了一定程度的隔离。  

* class **barbican.plugin.interface.secret_store.SecretDTO(type, secret, key_spec, content_type, transport_key=None)**  
机密数据传输对象（DTO）。  
此对象封装了key和它的属性。属性包括KeySpec，其中包含算法和长度。属性还包括有关密钥编码的信息。

* class **barbican.plugin.interface.secret_store.AsymmetricKeyMetadataDTO(private_key_meta=None, public_key_meta=None, passphrase_meta=None)**  
此对象封装了非对称密钥组件的元数据。  
这些组件包括private_key_meta，public_key_meta和passphrase_meta。  

## Secret Parameter Objects

秘密参数类封装了有关要存储在Barbican和/或其插件中的秘密的信息。  

* class **barbican.plugin.interface.secret_store.SecretType**  
用于定义对称密钥类型。  

由getSecret用来检索对称密钥。

* class **barbican.plugin.interface.secret_store.KeyAlgorithm**  
Diffie Hellman算法定义。

* class **barbican.plugin.interface.secret_store.KeySpec(alg=None, bit_length=None, mode=None, passphrase=None)**  
此对象指定密钥的算法和长度。

## Plugin Base Class  
Barbican机密存储插件应该实现抽象基类 **SecretStoreBase**。此插件的具体实现应该通过本指南配置部分中的 **stevedore**机制暴露给Barbican。  

class **barbican.plugin.interface.secret_store.SecretStoreBase**  
&nbsp;&nbsp;&nbsp;&nbsp; **delete_secret(secret_metadata)**  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;从机密存储中删除机密。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;机密一旦从机密存储中删除，它将不再被引用。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; **Parameters**  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; **secret_metadata** - 机密元数据  

&nbsp;&nbsp;&nbsp;&nbsp; **generate_asymmetric_key(key_spec)** 
&nbsp;&nbsp;&nbsp;&nbsp; 生成一对新的非对称密钥对并存储。  
&nbsp;&nbsp;&nbsp;&nbsp; 生成一对新的非对称密钥对并存储到机密存储中。AsymmetricKeyMetadataDTO类型的对象，包含了新创建的密钥对元数据属性，会被返回。元数据由Barbican存储并传递到其他方法以协助插件。这对于在外部数据存储中生成唯一ID并在将来使用它来检索密钥对的插件非常有用。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; **Parameters**  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; **key_spec** - 包含要生成的密钥类型的详细信息的KeySpec  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; **Returns**  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; AsymmetricKeyMetadataDTO类型的对象,包含了密钥对的元数据。  

&nbsp;&nbsp;&nbsp;&nbsp; **generate_supports(key_spec)**  
&nbsp;&nbsp;&nbsp;&nbsp; 返回bool值表明该机密类型是否支持。  
&nbsp;&nbsp;&nbsp;&nbsp; 生成方法检查算法和长度是否支持。这在调用generate_symmetric_key或者generate_asymmetric_key之前是很有用的，以便在生成之前建成密钥类型是否支持。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; **Parameters**  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; **key_spec** - 包含算法和长度细节的KeySpec。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; **Returns**  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; bool值指示算法是否支持。  

&nbsp;&nbsp;&nbsp;&nbsp; **generate_symmetric_key(key_spec)**  
&nbsp;&nbsp;&nbsp;&nbsp; 生成新的对称密钥并存储。  
&nbsp;&nbsp;&nbsp;&nbsp; 生成新的对称密钥并存储到机密存储中。包含新创建的对称密钥的元数据的字典会被返回。元数据字典由Barbican存储并传递到其他方法以协助插件。这对于在外部数据存储中生成唯一ID并在将来使用它来检索密钥对的插件非常有用。如果SecretStore不需要它，则返回的字典可能为空。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; **Parameters**  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; **key_spec** - 包含要生成的密钥类型的详细信息的KeySpec。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; **Returns**  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 包含此秘钥元数据的可选字典。  

&nbsp;&nbsp;&nbsp;&nbsp; **get_plugin_name()**  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 获取用户友好的插件名称。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 此插件的名字应从配置文件中读取。插件名会有一个默认值，如果有必要，在特定部署过程中可以自定义。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 部署中，此名称应该是唯一的。  

&nbsp;&nbsp;&nbsp;&nbsp; **get_secret(secret_type,secret_metadata)** 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 从机密存储中检索机密。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 从机密存储中检索机密并返回包含机密的SecretDTO。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 参数secret_metadata是生成或存储方法返回的元数据。这些数据在此插件中用来检机密。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 参数secret_type可以用于机密存储，用以了解机密的语气格式。例如，如果类型是SecretDTO.PRIVATE，则返回PKCS8结构。这样秘密商店就不需要自己管理秘密类型。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; **Parameters**  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; **secret_type** - 机密类型  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; **secret_metadata - 机密元数据  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; **Returns**  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 包含机密的SecretDTO。  

&nbsp;&nbsp;&nbsp;&nbsp; **get_transport_key()**  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 获取传输密钥  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 返回与插件有关的目前可用的传输密钥。传输密钥应该是包含公钥的base64编码的X509证书。管理员负责使用TransportKey资源上的DELETE方法从数据库中删除旧密钥。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 默认情况下，返回None。支持传输密钥包装的插件应覆盖此方法。  

&nbsp;&nbsp;&nbsp;&nbsp; **is_transport_key_current(transport_key)**  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;确定提供的传输密钥是不是当前的有效密钥。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 如果传输密钥是当前有效的传输密钥，则返回true。如果密钥无效，则barbican核心将从插件中请求新的传输密钥。   
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 默认返回False。支持传输密钥包装的插件应覆盖此方法。  

&nbsp;&nbsp;&nbsp;&nbsp; **store_secret(secret_dto)**  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 存储密钥  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SecretDTO包含机密的字节和机密的属性。SecretStore检索机密字节，存储它们，并返回关于机密的元数据字典。这对于在外部数据存储中生成唯一ID并在将来使用它来检索机密的插件非常有用。如果SecretStore不需要它，则返回的字典可能为空。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; **Parameters**  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; **secret_dto** 机密的SecretDTO  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; **Returns**  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 包含机密元数据的可选字典。  

&nbsp;&nbsp;&nbsp;&nbsp; **store_secret_supports(key_spec)**  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 返回bool值，指示机密是否可以存储。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 检查机密存储是否可以存储密钥。在KeySpec中提供秘密的属性。例如，一些插件可能需要知道属性以便存储秘密，但是如果没有给出属性，则其他插件可能能够将秘密存储为blob。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; **Parameters**  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; **key_spec** - 机密的KeySpec  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; **Returns**  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; bool值，标识机密可否被存储。  

## Barbican Core Plugin Sequence  

如下所述，在 **SecretStoreBase**中Barbican的调用顺序由请求操作决定。请注意，这些操作是通过 **barbican.plugin.resource**模块调用的，而模块又是通过Barbican的API和Worker进程调用。  

对于 **机密存储操作**，Barbican调用如下方法：  
1. **get_transport_key()** - 当上传机密进行存储时需要传输密钥，那么此方法要求插件提供传输密钥。  
2. **store_secret_supports()** - 询问插件是否可以支持基于 **KeySpec**参数信息存储密钥。  
3. **store_secret()** - 要求插件执行SecretDTO上面提供的未加密秘密有效负载的加密，然后存储该秘密。然后，该插件返回有关该秘密的信息字典（通常是对该插件唯一有意义的存储秘密的唯一引用）。然后Barbican核心将该字典作为其数据存储中的JSON属性保留，并将其交还给插件以便稍后进行秘密检索。用于执行此存储的插件的名称也由Barbican核心保留，以确保我们仅使用此插件检索此秘密。  

对于 **机密检索**，Barbican核心将选择与用于存储秘密的插件相同的插件，然后调用其**get_secret()**方法以返回未加密的秘密。  

对于 **对称密钥生成**，Barbican核心调用以下方法：  
1. **generate_supports()** - 询问插件是否可以支持基于 **KeySpec**如上所述的参数信息生成对称密钥。  
2. **generate_symmetric_key()** - 要求插件根据 **KeySpec**参数信息生成和存储对称密钥。然后插件可以返回存储的秘密的信息字典，类似于上面的存储过程，Barbican核心将持续存储以便稍后检索这个生成的秘密。  

对于 **非对称密钥生成**，Barbican核心调用以下方法：  
1. **generate_supports()** - 询问插件是否可以支持基于 **KeySpec**如上所述的参数信息生成非对称密钥。  
2. **generate_symmetric_key()** - 要求插件根据 **KeySpec**参数信息生成和存储非对称密钥。然后，插件可以返回 **AsymmetricKeyMetadataDTO**如上所述的对象，该对象包含由该插件生成和存储的三个秘密中的每一个的秘密元数据：私钥，公钥和可选的密码。然后Barbican核心将保留这些秘密的信息，并创建一个容器来对它们进行分组。  

## The Cryptographic Plugin Adapter  
Barbican核心包括一个专门的秘密商店插件，用于适应加密插件，称为**StoreCryptoAdapterPlugin**。此插件用作秘密存储插件，但它将秘密相关操作指向 加密插件以进行加密/解密/生成操作。由于加密插件不存储加密的机密，因此此适配器插件通过Barbican的数据存储提供此存储功能。  

此适配器插件还用于 **stevedore**访问和使用可支持秘密操作的加密插件。


# 加密插件开发  
本章节介绍如何开发供Barbican使用的自定义加密插件。  

Barbican支持两种机密存储模式：加密模式（本章节内容）和机密存储模式（上一章节内容）。加密模式在Barbican的数据存储中存储加密的机密，利用加密过程或设备（例如HSM）来执行加密/解密。Barbican提供了一个基于PKCS11的SafeNet HSM接口。  

请注意，加密插件不是从Barbican核心调用的，而是通过机密存储模式插件适配器类来调用。  

## crypto Module  
该 **barbican.plugin.crypto**模块包含实现自定义插件所需的类。这些类包括 **CryptoPluginBase**自定义插件应该继承的抽象基类，以及用于在Barbican和插件之间传输数据的多个数据传输对象（DTO）类。  

## Data Transfer Objects  
DTO类用来打包从Barbican传到插件的数据，也返回从插件到Barbican的数据。它们在插件和Barbican的内部数据模型之间提供了一定程度的隔离。  

* class **barbican.plugin.crypto.base.KEKMetaDTO(kek_datum)**  
&nbsp;&nbsp;&nbsp;&nbsp; 密钥加密密钥MetaDTO。  
&nbsp;&nbsp;&nbsp;&nbsp; Barbican中的密钥加密密钥（KEK）旨在标识用于对特定想的机密执行加密的不同密钥。  
&nbsp;&nbsp;&nbsp;&nbsp; **KEKMetaDTO** Barbican向加密后端提供的对象，以允许插件持久保存与香米的KEK相关的元数据。  
&nbsp;&nbsp;&nbsp;&nbsp; 例如，与HSM连接的插件可能希望每个项目使用不同的加密密钥。这样的插件可以使用该KEKMetaDTO对象来保存用于该项目的密钥ID。Barbican将持久保存KEK元数据，并确保每次处理来自同一项目的请求时都将其提供给插件。  
&nbsp;&nbsp;&nbsp;&nbsp; **plugin_name**  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Barbican用于标识绑定到KEK元数据的插件的字符串属性。插件不应该修改此属性。  
&nbsp;&nbsp;&nbsp;&nbsp; **kek_label** 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; string属性，用于通过插件标记项目的KEK。该属性的值对插件有意义。Barbican不适用此值。  
&nbsp;&nbsp;&nbsp;&nbsp; **algorithm**  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; string属性，用于标识插件使用的加密算法。例如AES、3DES等。此值对插件有意义，Barbican不使用。  
&nbsp;&nbsp;&nbsp;&nbsp; **mode**  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; string属性，用来标识插件使用的算法模式，例如CBC、GCM等。此值对插件有意义，Barbican不使用。  
&nbsp;&nbsp;&nbsp;&nbsp; **bit_length** 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 整数属性，用于通过插件标识KEK的长度。此值对插件有意义，Barbican不使用。  
&nbsp;&nbsp;&nbsp;&nbsp; **plugin_meta**  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; string属性，用于保留任何不适合任何其他属性的其他元数据。该属性的值由插件定义。它可用于存储外部系统引用，例如HSM中的密钥ID，外部服务的URI或插件认为必须保留的任何其他数据。因为这只是一个纯文本字段，所以插件甚至可以选择在JSON对象中保存诸如键值对之类的数据。  

* class **barbican.plugin.crypto.base.EncryptDTO(unencrypted)**  
&nbsp;&nbsp;&nbsp;&nbsp; 机密加密DTO。  
&nbsp;&nbsp;&nbsp;&nbsp; 数据传输对象用于传递插件执行秘密加密所需的所有数据。  
&nbsp;&nbsp;&nbsp;&nbsp; 目前，此DTO仅包含要由插件加密的原始字节，但将来可能包含更多信息。  
&nbsp;&nbsp;&nbsp;&nbsp; **unencrypted**  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 要通过插件加密的字节秘密数据。 

* class **barbican.plugin.crypto.base.DecryptDTO(encrypted)**  
&nbsp;&nbsp;&nbsp;&nbsp; 机密解密DTO。 
&nbsp;&nbsp;&nbsp;&nbsp; 数据传输对象用于传递插件执行秘密解密所需的所有数据。  
&nbsp;&nbsp;&nbsp;&nbsp; 目前，此DTO仅包含插件在加密期间生成的数据，但将来此DTO将包含更多信息，例如用于秘密回送到客户端的传输密钥。  
&nbsp;&nbsp;&nbsp;&nbsp; **encrypted**  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;加密期间插件生成的数据。对于某些插件，这将是需要解密以产生秘密的实际字节。在其他实现中，这可以仅仅是对可以产生未加密秘密的一些外部系统的引用。  

* class **barbican.plugin.crypto.base.GenerateDTO（algorithm，bit_length，mode，passphrase = None)**  
&nbsp;&nbsp;&nbsp;&nbsp; 机密生成DTO  
&nbsp;&nbsp;&nbsp;&nbsp; 数据传输对象用于传递插件的所有必要数据以代表用户生成机密。  
&nbsp;&nbsp;&nbsp;&nbsp; **generation_type**  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; string属性，用于标识生成机密的类型，可选“**symmetric**”和“**asymmetric**”。  
&nbsp;&nbsp;&nbsp;&nbsp; **algorithm**  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; string属性，用于标识机密用于哪种算法，例如，“**AES**”对应某种“**symmetric**”，“**RSA**"对应”**asymmetric**”。  
&nbsp;&nbsp;&nbsp;&nbsp; **mode**  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; string属性，用于指定机密将用于哪种算法模式，例如，”**CBC**“用于”**AES**”算法。  
&nbsp;&nbsp;&nbsp;&nbsp; **bit_length**  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; int属性，用于执行机密的长度。  

## Plugin Base Class  
Barbican机密存储插件应该实现抽象基类 **CryptoPluginBase**。此插件的具体实现应该通过本指南配置部分中的 **stevedore**机制暴露给Barbican。 

* class **barbican.plugin.crypto.base.CryptoPluginBase**  
&nbsp;&nbsp;&nbsp;&nbsp; 所有加密插件的基类。  
&nbsp;&nbsp;&nbsp;&nbsp; Barbican通过在实现类的实例上调用方法来请求操作。Barbican的插件管理器处理传递给这些方法的数据传输对象（DTO）的生命周期，并持久保存插件分配给这些DTO的数据。  

&nbsp;&nbsp;&nbsp;&nbsp; **bind_kek_metadata(kek_meta_dto)**  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 密钥机密密钥元数据绑定功能。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 将密钥加密密钥（KEK）元数据绑定到处理加密/解密的子系统，更新所提供的'kek_metadata'数据传输对象实例中的密钥加密密钥（KEK）元数据的信息，然后返回该实例。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 在上面的encrypt（）方法之前调用此方法。实现者应根据需要填写提供的'kek_meta_dto'实例（上面的KEKMetadata实例），以完整描述kek元数据并完成绑定过程。一旦此方法返回，Barbican将保留此实例的内容。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; **Parameters**  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; **kek_meta_dto** - 要绑定的密钥加密密钥元数据，'kek_label'属性保证是唯一的，并且'plugin_name'属性已经配置。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; **Returns**  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; kek_meta_dto：修改后返回指定的DTO。  

&nbsp;&nbsp;&nbsp;&nbsp; **decrypt(decrypt_dto，kek_meta_dto，kek_meta_extended，project_id)**  
&nbsp;&nbsp;&nbsp;&nbsp;在提供的项目的上下文中解密decrypt_dto.
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; **Parameters**:  
* &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; decrypt_dto - 包含要解密的密文的DTO  
* &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; kek_meta_dto - 用于解密的密钥加密密钥元数据  
* &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; kek_meta_extended - 可选项，用于解密的每个机密的KEK  
* &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; project_id - 与加密数据相关的项目ID  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; **Returns**  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; string类型，未加密的数据  

&nbsp;&nbsp;&nbsp;&nbsp; **encrypt(encrypt_dto,kek_meta_dto,priject_id)**   
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 加密函数。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 项目中，当要求对机密进行加密操作时，Barbican将调用此方法。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; **Parameters**  
* &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; encrypt_dto - 要加密的包含原始机密数据的EncryptDTO的实例。  
* &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; kek_meta_dto - 用于加密的包含项目KEK信息的KEKMetaDTO实例。插件可能会假定bind_kek_metadata()在传递此实例之前已经发生了绑定过程 。  
* &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; project_id - 与未加密数据相关的项目ID  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; **Returns**  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 包含密文和KEK信息的DTO  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; **Return Type**  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ResponseDTO  

&nbsp;&nbsp;&nbsp;&nbsp; **generate_asymmetric(generate_dto,kek_meta_dto,project_id)**  
&nbsp;&nbsp;&nbsp;&nbsp; 创建一个新的非对称密钥。  
&nbsp;&nbsp;&nbsp;&nbsp; **Parameters**  
* &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; generate_dto - 与此生成请求关联的记录的数据传输对象。可以从该对象中提取一些相关参数，包括长度，算法和模式。  
* &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; kek_meta_dto - 用于解密的KEK元数据。  
* &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; project_id - 与数据关联的项目ID  

&nbsp;&nbsp;&nbsp;&nbsp; **generate_symmetric(generate_dto,kek_meta_dto,project_id)**  
&nbsp;&nbsp;&nbsp;&nbsp; 创建一个新的对称密钥。  
&nbsp;&nbsp;&nbsp;&nbsp; **Parameters**  
* &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; generate_dto - 与此生成请求关联的记录的数据传输对象。可以从该对象中提取一些相关参数，包括长度，算法和模式。  
* &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; kek_meta_dto - 用于解密的KEK元数据。  
* &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; project_id - 与数据关联的项目ID   

&nbsp;&nbsp;&nbsp;&nbsp; **get_plugin_name()**  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 获取用户友好的插件名称。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 此插件的名字应从配置文件中读取。插件名会有一个默认值，如果有必要，在特定部署过程中可以自定义。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 部署中，此名称应该是唯一的。  

&nbsp;&nbsp;&nbsp;&nbsp; **supports(type_enum,algorithm=None,bit_length=None,mode=None)**  
&nbsp;&nbsp;&nbsp;&nbsp; 用于确定插件是否支持请求的操作。  
&nbsp;&nbsp;&nbsp;&nbsp; **Parameters**  
* &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; **type_enum** - PluginSupportsType类中的枚举值。  
* &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; **algorithm** - 可选值，算法名称。  

## Barbican Core Plugin Sequence  
Barbican根据不同的请求操作调用 **CryptoPluginBase**插件的不同方法。请注意，这些操作是通过秘密存储适配器类调用的，该类**StoreCryptoAdapterPlugin**在“加密插件适配器”中进一步描述。

对 **存储操作**，Barbican调用如下方法：  
1. **supports()** - 查询插件是否支持 **barbican.plugin.crypto.base.PluginSupportTypes.ENCRYPT_DECRYPT**操作类型。  
2. **bind_kek_metadata()** - 允许插件将内部密钥加密密钥（KEK）绑定到项目ID，通常作为“标签”或对加密设备中存储的实际KEK的引用。此KEK信息代表插件存储在Barbican的数据存储中，然后提供回插件以供后续调用。  
3. **encrypt()** - 要求插件使用绑定到上面项目ID的KEK对未加密的秘密有效负载进行加密。然后Barbican核心将持久保存从此方法返回的加密数据，以便以后检索。用于执行此加密的插件的名称也会持久保存到Barbican核心，以确保我们仅使用此插件解密此秘密。  

对于 **机密解密和检索**，Barbican核心将选择与用于存储秘密的插件相同的插件，然后调用其 **decrypt()**方法，为其提供先前持久的加密机密数据以及用于加密机密的项目ID KEK。  

对于 **对称密钥生成**，Barbican调用以下方法：  
1. **supports()** - 查询插件是否支持 **barbican.plugin.crypto.base.PluginSupportTypes.SYMMETRIC_KEY_GENERATION**操作类型。  
2. **bind_kek_metadata()** - 同 **存储操作**第2步说明一样。  
3. **generate_symmetric()** - 请求插件，生成一个对称密钥，并使用项目ID对应KEK加密它。  

对于 **非对称密钥生成**，Barbican调用以下方法：  
1. **supports()** - 查询插件是否支持 **barbican.plugin.crypto.base.PluginSupportTypes.ASYMMETRIC_KEY_GENERATION**操作类型。  
2. **bind_kek_metadata()** - 同 **存储操作**第2步说明一样。  
3. **generate_asymmetric()** - 请求插件，生成并使用项目ID对应KEK加密非对称公钥和私钥（和可选的密码）信息。


