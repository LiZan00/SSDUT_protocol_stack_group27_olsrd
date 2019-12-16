##  Abstract

+ OLSR 是优化链路状态路由协议，基于多点中继MPR
+ 经典的洪泛机制是每个节点在第一次收到该消息的第一个副本时重传每条信息，而MPR与之相比减小了信息开销。
+ 在OLSR中，链路信息仅会由被选为MPR的节点生成
+ OLSR的第二个优点是减少了控制信息control messages在网络中洪泛的数量。
+ 第三个优点是，一个MPR节点可能会选择仅报告其自身与选它作为MPR节点的节点之间的连接。
+  OLSR与经典的链路状态路由算法相反，部分链路状态信息分布在网络中。这些信息接下来会被用于路由计算
+  OLSR提供最佳路由（最佳跳数），适合大型，密集网络



## 1 Introduction

+ OLSR的开发是为了移动自组织网络，他是表驱动的，主动协议，即与其他节点定期交换拓扑信息
+ 每个节点选择一组其邻居节点作为“多点中继”（MPR），在OLSR中，只有被选为MPR的节点负责转发控制信息，旨在用于扩散到整个网络。MPR提供一个有效的机制，这个机制可以通过减少传输请求的数量来flooding控制信息 。
+ 被选为MPR的节点要在网络中声明链路状态信息，OLSR提供到所有目的地的最短路径的功能就是依靠MPR节点声明选其作为MPR节点的节点的链接状态信息来实现的。
+ 已被某些邻居节点选为MPR的节点在其控制消息中周期性地宣布该信息，就相当于这个MPR节点向大家宣布自己和选他作为MPR的节点是联通的，在路由计算中，MPR被用于形成从给定节点到网络中其他所有节点的路由。此外，该协议使用MPR来促进网络中控制消息的有效泛滥。
+ 节点是从自己周围一跳的邻居里选择MPR节点

### 1.1 术语
+ OLSR 接口： 运行OLSR协议的接口，一个路由器可以有好多接口，每个接口一个ip地址
+ Non OLSR 接口：和上面相反
+ single OLSR interface node
+ multiple OLSR interface node
+ 主要地址：有多个olsr接口的节点需要选一个出来作为主要地址
+ MPR： 是一个节点，被他的一跳邻居X选作MPR，这个MPR负责重传他从X收到的所有广播信息，前提是这个信息不是重复的而且TTL>1
+ MPR selector：X选择Y作为MPR，X是Y的MPR selector
+ symmetric link（对称连接）两个OLSR接口经过双向验证的连接
+ asymmetric link（非对称连接）仅在一个接口（方向）中进行了验证
+ 对称邻居：至少有一个对称连接。

### 1.2 适用性
OLSR适合大型，密集的网络
使用逐跳路由（hop-by-hop routing）

### 1.3 协议overview
+ OLSR分布式工作
+ 不需要按顺序传输信息，每条信息都有一个序列标号
+ 支持扩展
+ 不需要对ip数据包格式进行修改
+ （球球作者别吹了）

### 1.4 MPR
+ 多点中继的思想是通过减少同一区域内的冗余重传来最小化网络中的泛洪消息开销。
+ 节点N的不是其MPR的邻居可以接收处理广播信息但是不能重传广播信息。
+ 每个节点从其1跳对称邻居中选择其MPR集。这个集合要覆盖所有严格对称的2跳节点，N的MPR集表示为MPR（N），它是N的所有对称1跳邻居的任意子集，满足以下条件：N的严格对称2跳邻居中的每个节点都必须有指向N的对称链路。MPR集越小，路由协议产生的控制流量开销越小。
（他的意思应该是N要是想要和对称二跳邻居通信，必须要经过他MPR集中的MPR）
+ 每个节点维护将他选为MPR的节点的信息。节点从邻居接收到的周期性HELLO消息中获取此信息。
+ 如果N尚未接收到来自任意一个选择N作为MPR节点的节点的、打算在整个网络中扩散的广播消息，则假定该广播消息由节点N重新发送。（我猜这句话是想说，当MPR节点N收到一条广播信息时，这条信息如果没有被任何将他选为MPR的节点转发过，那么N就假定这个消息由他转发。）





## 2 协议功能

+ 核心功能是在独立的移动自组织网络中提供路由
+ 辅助功能可兼容

### 2.1 核心功能

+ 数据包格式和转发

+ 链路检测 （link sensing）

  + 链路检测是通过在接口上周期性地发送HELLO消息来完成的，每个接口生成单独的HELLO消息，并按照特定规则发出。

  + 链路检测会生成一个本地的连接状态集合，这个集合用来描述当前接口和他的邻居接口的连接状态
  + 如果链路层能提供足够的信息，就不需要用交换HELLO 消息来生成连接状态集

+ 邻居检测（监听）

  + 对于所有节点都是单接口的网络来说，每个节点的唯一的接口的地址就是主要地址（main address）
  + 在具有多接口节点的网络中，需要附加信息才能将接口地址映射到主地址（从而映射到节点）。附加信息通过”多接口声明“（multiple interface declaration）（MID）消息获取。

+ MPR选择和MPR Signaling

  + 节点从所有邻居里选出一个子集，子集里的节点叫该节点的MPR节点。
  + 节点发出的消息通过其MPR节点转发以后，理论上因该被所有据它两跳及以外的节点所接收
  + 选择及更新MPR集也是通过HELLO 信息来确定的

+ 拓扑控制信息的传播（Diffusion）
  + 为了向网络中的每个节点提供足够的链路状态信息以允许路由计算，对拓扑控制消息以特定的规定进行了传播。
+ 路由计算
  + 给定通过周期性消息交换获得的链路状态信息，以及节点的接口配置，可以计算出每个节点的路由表

### 2.2 辅助功能

+ 包括一个节点有多个接口需要选择主要地址的情况等



## 6 HELLO消息的格式和生成

### 6.1 HELLO消息的格式

![img](file:///C:\Users\owo\AppData\Roaming\Tencent\Users\2630195018\QQ\WinTemp\RichOle\EM[}JV}4{YNN0ALT$R__S4V.png)

+ 1.Reserved （保留字段）

  + 此字段必须设置为“0000000000000”
+ 2.HTime

  + 此字段指定节点在此特定接口上使用的HELLO发射时间间隔，即此HELLO发送到下一个HELLO的发送之前的时间，HELLO发射间隔由其尾数（Htime字段的四个最高位）和指数（Htime字段的四个最低位）表示。
+  Willingness
  + 此字段指定节点是否愿意为其他节点结转和转发流量。
  + 一个Willingness被设置成 WILL_NEVER  的节点决不能被任意节点选作MPR
  + 一个Willingness被设置成 WILL_ALWAYS  的节点必须永远被选作MPR
  + 一个节点应该公布其Willingness的值
+  Link Code
  + 此字段说明了发送方接口和邻居接口列表之间的链接的信息。它还说明了邻居的状态信息
  + 节点看不懂的LinkCode会被自动丢弃
+  Link Message Size
  + 此字段为链路信息的大小，以字节为单位，为一个"Link Code"字段到下一个"Link Code"字段之间的长度
+ Neighbor Interface Address （邻居接口地址）
  + 邻居节点的接口地址。

#### 6.1.1 Link Code as Link Type and Neighbor Type

+ Link Code字段的格式

  ![img](file:///C:\Users\owo\AppData\Roaming\Tencent\Users\2630195018\QQ\WinTemp\RichOle\MD`[}YSTX$M741E1@Q%0}8G.png)

+ OLSR规定的链路类型（link type）有以下四种

  +  UNSPEC_LINK ： 表示没有给出关于连接的特定信息（unspecified）
  + ASYM_LINK ： 表示链路是非对称的
  + SYM_LINK： 表示链路和接口是对称的
  + LOST_LINK： 连接丢失

+ OLSR规定的邻居类型（Neighbor Type）有以下三种

  + SYM_NEIGH： 表示邻居至少有一个关于此节点的对称链接。
  + MPR_NEIGH： 表示邻居具有至少一个对称连接并且已被发送方选择为MPR。
  + NOT_NEIGH： 无对称邻居
  
+ 链路状态和邻居状态不能再逻辑上有冲突，比如

   Link Type     == SYM_LINK AND  Neighbor Type == NOT_NEIGH 就讲不通

### 6.2 HELLO 消息的生成

+ HELLO 消息提供三个独立任务
  + link sensing（链路检测）
  + neighbor detection（邻居检测）
  +  MPR selection signaling
+ 这三个任务都是根据周期性信息交换实现的，HELLO 信息是由本地链路集、邻居集和MPR集中的信息生成的
+ 节点必须在每个接口上执行链路检测，以便检测接口和邻居接口之间的链路。此外，为了执行邻居检测，节点必须在每个接口上公布其所有与之对称的1跳邻居。因此，对于给定的接口，HELLO消息将包含该接口上的链接列表以及所有邻居的列表

+ HELLO消息中声明的邻居接口地址列表是用如下方式计算的

  + ```java
      1    The Link Type set according to the following:
      
              1.1  if L_SYM_time >= current time (not expired)
      
                        Link Type = SYM_LINK
      
              1.2  Otherwise, if L_ASYM_time >= current time (not expired)
                   AND
      
                                 L_SYM_time  <  current time (expired)
      
                        Link Type = ASYM_LINK
                   1.3  Otherwise, if L_ASYM_time < current time (expired) AND
          
                                   L_SYM_time  < current time (expired)
          
                          Link Type = LOST_LINK
          
           2    The Neighbor Type is set according to the following:
          
                2.1  If the main address, corresponding to
                     L_neighbor_iface_addr, is included in the MPR set:
          
                          Neighbor Type = MPR_NEIGH
          
                2.2  Otherwise, if the main address, corresponding to
                     L_neighbor_iface_addr, is included in the neighbor set:
          
                     2.2.1
                          if N_status == SYM
          
                               Neighbor Type = SYM_NEIGH
          
                     2.2.2
                          Otherwise, if N_status == NOT_SYM
                               Neighbor Type = NOT_NEIGH
                              （不翻译了，反正就是这个意思）
      ​```
        
      ```



+ HELLO消息可以分片发送，规则是在每一个刷新周期里，一个节点的链路信息和邻居节点至少被HELLO 信息发送过一次

### 6.3 HELLO 消息的转发

+ 每个HELLO消息都由节点在一个接口上广播给其邻居，但是HELLO消息绝对不允许被转发

### 6.4 HELLO消息的处理

+ 节点处理传入的HELLO消息以进行链路感知、邻居检测和MPR选择器集填充



## 7 Link Sensing （链路检测/监听）

+ 链路检测生成local link information set。链路检测只关心OLSR接口地址和这些OLSR接口之间交换数据包的能力

### 7.1Populating the Link Set 生成链路集

+  Link Set （链路集）包含的信息是到邻居节点的链路信息，填充链路集的过程被称为链路检测，这个过程通过交换HELLO信息来执行
+ 每个节点都应该检测自己和邻居节点之间的链接。无线电传播的不确定性可能会使某些链路成为单向的。因此，双向可达的连接才能被认为是有效连接
+ “链接”这个词由本地接口和远程接口描述（如何描述一条线段？ 标明起点和终点）
+ “对称链路”表示，到该邻居节点的链路已被验证为双向的，即，可以在两个方向上传输数据。
+ “非对称链路”表示已听到来自节点的HELLO消息（即，可以从邻居节点进行通信），但未确认该节点也能够接收消息（即，未确认与邻居节点的通信）。
+ 通过链路检测获得的信息被放在链路集（Link Set）里

#### 7.1.1 HELLO 消息的处理

+ HELLO 消息的发送端地址（"Originator Address" ）就是节点的主地址

+ 收到HELLO消息后，节点应更新其链路集（link set）。请注意，HELLO消息既不能转发，也不能被重复记录

+ 接收到HELLO消息后，必须从消息头的Vtime字段计算“有效时间”（validity time） 。然后，链接集应按照如下规则更新：

  ```
   1    Upon receiving a HELLO message, if there exists no link tuple
            with
  
                 L_neighbor_iface_addr == Source Address
  
            a new tuple is created with
  
                 L_neighbor_iface_addr = Source Address
  
                 L_local_iface_addr    = Address of the interface
                                         which received the
                                         HELLO message
  
                 L_SYM_time            = current time - 1 (expired)
  
                 L_time                = current time + validity time
  
       2    The tuple (existing or new) with:
  
                 L_neighbor_iface_addr == Source Address
  
            is then modified as follows:
  
            2.1  L_ASYM_time = current time + validity time;
  
            2.2  if the node finds the address of the interface which
                 received the HELLO message among the addresses listed in
                 the link message then the tuple is modified as follows:
  
  
  
  Clausen & Jacquet             Experimental                     [Page 34]
  
  RFC 3626              Optimized Link State Routing          October 2003
  
  
                 2.2.1
                      if Link Type is equal to LOST_LINK then
  
                           L_SYM_time = current time - 1 (i.e., expired)
  
                 2.2.2
                      else if Link Type is equal to SYM_LINK or ASYM_LINK
                      then
  
                           L_SYM_time = current time + validity time,
  
                           L_time     = L_SYM_time + NEIGHB_HOLD_TIME
  
            2.3  L_time = max(L_time, L_ASYM_time)
  ```



## 8 邻居检测

+ 邻居检测生成，填充邻居信息
+ 这玩意也是依靠交换HELLO信息来实现的

### 8.1 Populating the Neighbor Set 填充邻居集

+ 节点持有一个邻居集，这个集合根据链路集进行更新

+ 生成

  + 每次出现链接时，也就是说，每当创建链接元组（link tuple）时，必须创建关联的邻居元组（neighbor tuple）（邻居元组的集合形成邻居集），如果它不存在，则具有以下值

    N_neighbor_main_addr = main address of
                                          L_neighbor_iface_addr
                                          (from the link tuple)

+ 更新

  + 每次链接更改时，即每次修改链接元组的信息时，节点必须确保关联的邻居元组的N_status符合以下属性：

    如果邻居有任何关联的链接元组，该元组描述的链接是对称的（即，L_SYM_time>=当前时间），则 N_status is set to SYM，

    ​											否则N_status is set to NOT_SYM

+ Removal 

  + 每次删除链接时，即每次删除链接元组时，如果关联的邻居元组不再具有任何关联的链接元组，则必须将其删除。
  + 这些规则确保链接元组只有一个关联的邻居元组，并且每个邻居元组至少有一个关联的链接元组。

#### 8.1.1 HELLO 消息的处理

+ 在接收到HELLO消息时，节点应该首先更新其链接集，如前所述。然后，它应该更新其邻居集，如下所示：

  + 如果HELLO消息的原地址是邻居集中包含的邻居元组的N_neighbor_main_addr：

    邻居元组应该如下更新： N_willingness = willingness from the HELLO message

### 8.2  填充2跳邻居集

+ 和节点N的对称邻居具有对称链接的节点成为N的两跳邻居

#### 8.2.1 HELLO消息的处理

+ 当从对称邻居接收到HELLO消息时，节点应该更新其2跳邻居集。

+ 如果发送HELLO消息的节点为其对称邻居，两跳邻居集应该如下更新

  + ```
     1    for each address (henceforth: 2-hop neighbor address), listed
              in the HELLO message with Neighbor Type equal to SYM_NEIGH or
              MPR_NEIGH:
      
              1.1  if the main address of the 2-hop neighbor address = main
                   address of the receiving node:
      
                        silently discard the 2-hop neighbor address.
      
                   (in other words: a node is not its own 2-hop neighbor).
      
              1.2  Otherwise, a 2-hop tuple is created with:
      
                        N_neighbor_main_addr =  Originator Address;
      
                        N_2hop_addr          =  main address of the
                                                2-hop neighbor;
      
                        N_time               =  current time
                                                + validity time.
     This tuple may replace an older similar tuple with same
                  N_neighbor_main_addr and N_2hop_addr values.
       
        2    For each 2-hop node listed in the HELLO message with Neighbor
             Type equal to NOT_NEIGH, all 2-hop tuples where:
       
                  N_neighbor_main_addr == Originator Address AND
       
                  N_2hop_addr          == main address of the
                                          2-hop neighbor
       
             are deleted.
     ```
  
  
  
  

### 8.3 填充MPR集

+ MPR是每个接口计算的，每个接口的MPR集的并集构成节点的MPR集。
+ MPR集越小，协议开销越小

#### 8.3.1 MPR Computation （MPR计算）

+ “neighbor of an interface” ：如果接口（在本地节点上）具有到该相邻节点的任何一个接口的链接，则该节点是“接口的邻居”。

+ N：N是节点邻居的子集，它是接口I的邻居

+ N2：可从接口I访问的两跳邻居集，不包括：

  + 只能从N中WILL_NEVER节点访问到的二跳邻居

  + 执行计算的节点

  + 所有对称邻居：在某个接口上存在与该节点对称链接的节点。

    （既是一跳邻居也是二跳邻居的节点）

+ D(y): 一跳邻居节点y（其中y是N的成员）的度定义为节点y的对称邻居的数目，不包括N的所有成员，也不包括执行计算的节点。

+ 如下计算

+ ```
   选择MPR的算法
   1    Start with an MPR set made of all members of N with
            N_willingness equal to WILL_ALWAYS
    
       2    Calculate D(y), where y is a member of N, for all nodes in N.
    
       3    Add to the MPR set those nodes in N, which are the *only*
            nodes to provide reachability to a node in N2.  For example,
            if node b in N2 can be reached only through a symmetric link
            to node a in N, then add node a to the MPR set.  Remove the
            nodes from N2 which are now covered by a node in the MPR set.
            如果有这样的N2节点：他只能通过某个N节点到达远点，那么这个N节点就是MPR，重复这个过程
    
       4    While there exist nodes in N2 which are not covered by at
            least one node in the MPR set:
            如果现在N2节点还没有被MPR覆盖全的话
             4.1  For each node in N, calculate the reachability, i.e., the
                number of nodes in N2 which are not yet covered by at
                least one node in the MPR set, and which are reachable
                through this 1-hop neighbor;
                对于N中的每个节点，计算可达性，即N2中尚未被MPR集中的至少一个节点覆盖并且可通过该1跳邻					居到达的节点的数目；
     
           4.2  Select as a MPR the node with highest N_willingness among
                the nodes in N with non-zero reachability.  In case of
                multiple choice select the node which provides
                reachability to the maximum number of nodes in N2.  In
                case of multiple nodes providing the same amount of
                reachability, select the node as MPR whose D(y) is
                greater.  Remove the nodes from N2 which are now covered
                by a node in the MPR set.
                在非零可达性的N个节点中，选择N中意愿最高的节点作为MPR。在多选择的情况下，选择在N2中的节点的最大数量提供可达性的节点。如果多个节点提供相同的可达性，则选择D（y）较大的节点作为MPR。从现在由MPR集中的节点覆盖的N2中移除节点。
     
      5    A node's MPR set is generated from the union of the MPR sets
           for each interface.  As an optimization, process each node, y,
           in the MPR set in increasing order of N_willingness.  If all
           nodes in N2 are still covered by at least one node in the MPR
           set excluding node y, and if N_willingness of node y is
           smaller than WILL_ALWAYS, then node y MAY be removed from the
           MPR set.
   ```

​       

 ### 8.4     Populating the MPR Selector Set （MPR Selector参照第一张定义，这个定义很坑）

+ 节点的MPR Selector Set n由选择n作为MPR的节点的主地址填充。

#### 8.4.1 处理HELLO 消息

+ 在接收到HELLO消息时，如果节点在列表中找到自己的接口地址之一，并且邻居类型等于MPR_NEIGH，则来自HELLO消息的信息必须记录在MPR选择器集中。（说明自己被别人当作了MPR）

+ 更新如下  



     1    If there exists no MPR selector tuple with:                
                     MS_main_addr   == Originator Address
      
                 then a new tuple is created with:
      
                      MS_main_addr   =  Originator Address
      
       2    The tuple (new or otherwise) with
      
                 MS_main_addr   == Originator Address
      
            is then modified as follows:
      
                 MS_time        =  current time + validity time.
                 
+ MPR Selector 元组的删除发生在计时器过期或“邻域和2跳邻域更改”中描述的链路断开的情况下。

### 8.5 邻居和两跳邻居变化Neighborhood and 2-hop Neighborhood Changes

+ 当发生以下情况时，检测到邻居发生了变化：

  + 一个链接元组过期（L_SYM_time超时），如果这是两个节点之间的最后一个链接，那么这个邻居就应该被删除
  + 将新的链接元组插入到具有未过期L_SYM_time的链接集中，或者修改具有已过期L_SYM_time的元组，使其L_SYM_time变为未过期。如果以前没有这个链接对应的邻居，那么添加邻居。

+ 当检测到邻居或2跳邻居中的变化时，发生以下处理：

  + 如果邻居丢失，则所有两跳邻居元组with
              N_neighbor_main_addr == Main Address of the neighbor 

    必须被删除

  + 在邻居丢失的情况下，所有MPR选择器元组with

     MS_main_addr == Main Address of the neighbor MUST be deleted

  + 当检测到邻居出现或丢失，或者当检测到2跳邻居中的变化时，必须重新计算MPR集。

  + 当MPR集改变时，可能会发送额外的HELLO消息。

  

  