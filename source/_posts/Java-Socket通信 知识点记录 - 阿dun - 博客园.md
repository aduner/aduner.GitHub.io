---
title: Java-Socket通信 知识点记录
date: 2021-03-04 18:00
categories: ['Java']
tags: ['套接字', 'Socket', 'Java']
---
#  一、Socket基本案例

  * **Server端**

    
    
    package demo1;
    
    import java.io.IOException;
    import java.io.InputStream;
    import java.net.ServerSocket;
    import java.net.Socket;
    
    public class SocketServer {
        public static void main(String[] args) throws IOException {
            int port = 47639;
            ServerSocket server = new ServerSocket(port);
    
            //等待连接
            System.out.println("server waits for connection...");
            Socket socket = server.accept();
            // 建立连接后，从socket中获取输入流，并建立缓冲区进行读取
            InputStream inputStream = socket.getInputStream();
            byte[] bytes = new byte[1024];
            int len;
            StringBuffer stringBuffer = new StringBuffer();
            while ((len = inputStream.read(bytes)) != -1) {
                //编码格式要统一
                stringBuffer.append(new String(bytes, 0, len, "UTF-8"));
            }
            System.out.println("get message from client: " + stringBuffer);
            inputStream.close();
            socket.close();
            server.close();
        }
    
    }
    
    

  * **Client端**

    
    
    package demo1;
    
    import java.io.IOException;
    import java.io.OutputStream;
    import java.net.Socket;
    import java.nio.charset.StandardCharsets;
    
    public class SocketClient {
        public static void main(String[] args) throws IOException {
            // 要连接的服务端IP地址和端口
            String host="127.0.0.1";
            int port=47639;
            // 与服务端建立连接
            Socket socket=new Socket(host,port);
            // 建立连接后获得输出流
            OutputStream outputStream=socket.getOutputStream();
            String message="hello world";
            System.out.println("send "+host+port+": "+message);
            //注意编码一致
            socket.getOutputStream().write(message.getBytes(StandardCharsets.UTF_8));
            outputStream.close();
            socket.close();
        }
    }
    
    

Server端终端打印信息如下

    
    
    server waits for connection...
    get message from client: hello world
    

#  二、消息通信

##  2.1 双向通信

在前例的基础上简单修改即可实现 **双向通信** 。

**Server**

当读取完 **客户端的消息后，打开输出流** ，发送数据即可

    
    
    package demo2;
    
    import java.io.InputStream;
    import java.io.OutputStream;
    import java.net.ServerSocket;
    import java.net.Socket;
    
    public class S {
        public static void main(String[] args) throws Exception {
            // 监听指定的端口
            int port = 47639;
            ServerSocket server = new ServerSocket(port);
            //等待连接
            System.out.println("server waits for connection...");
            Socket socket = server.accept();
            // 建立连接后，从socket中获取输入流，并建立缓冲区进行读取
            InputStream inputStream = socket.getInputStream();
            byte[] bytes = new byte[1024];
            int len;
            StringBuffer stringBuffer = new StringBuffer();
            while ((len = inputStream.read(bytes)) != -1) {
                //编码格式要统一
                stringBuffer.append(new String(bytes, 0, len, "UTF-8"));
            }
            System.out.println("get message from client: " + stringBuffer);
            System.out.println("send message to client");
            //回复
            OutputStream outputStream = socket.getOutputStream();
            outputStream.write("Hello Client,I get the message.".getBytes("UTF-8"));
    
            inputStream.close();
            outputStream.close();
            socket.close();
            server.close();
        }
    }
    

**Client端** 同理，只需要打开输入流即可

    
    
    package demo2;
    
    import java.io.IOException;
    import java.io.InputStream;
    import java.io.OutputStream;
    import java.net.ServerSocket;
    import java.net.Socket;
    import java.nio.charset.StandardCharsets;
    
    public class C {
        public static void main(String[] args) throws IOException {
            // 要连接的服务端IP地址和端口
            String host = "127.0.0.1";
            int port = 47639;
            // 与服务端建立连接
            Socket socket = new Socket(host, port);
            // 建立连接后获得输出流
            OutputStream outputStream = socket.getOutputStream();
            String message = "hello world";
            System.out.println("send " + host + port + ": " + message);
            //注意编码一致
            socket.getOutputStream().write(message.getBytes(StandardCharsets.UTF_8));
            //通过不再输入数据，后续只能接受数据
            socket.shutdownOutput();
            InputStream inputStream = socket.getInputStream();
            byte[] bytes = new byte[1024];
            int len;
            StringBuffer stringBuffer = new StringBuffer();
            while ((len = inputStream.read(bytes)) != -1) {
                //编码格式要统一
                stringBuffer.append(new String(bytes, 0, len, "UTF-8"));
            }
            System.out.println("get massage from server: " + stringBuffer);
            inputStream.close();
            outputStream.close();
            socket.close();
    
        }
    }
    

Server端终端打印信息如下

    
    
    server waits for connection...
    get message from client: hello world
    send message to client
    

Client端终端打印信息如下

    
    
    send 127.0.0.147639: hello world
    get massage from server: Hello Client,I get the message.
    

##  2.2 告知发送结束

正常来说，客户端打开一个输出流，如果不做约定，也不关闭它，那么服务端永远不知道客户端是否发送完消息，那么服务端会一直等待下去，直到读取超时。

###  2.2.1 通过Socket关闭

当Socket关闭的时候，服务端就会收到响应的关闭信号，那么服务端也就知道流已经关闭了，这个时候读取操作完成，就可以继续后续工作。

**缺点：**

  * 客户端Socket关闭后，将不能接受服务端发送的消息，也不能再次发送消息 
  * 如果客户端想再次发送消息，需要重现创建Socket连接 

###  2.2.2 通过Socket关闭输出流的方式

    
    
    socket.shutdownOutput(); //正确用法
    
    
    
    outputStream.close(); //错误用法
    

  * 方法二关闭了输出流，那么相应的 ` Socket ` 也将关闭，和直接关闭 ` Socket ` 一个性质。 

  * 调用 ` Socket ` 的 ` shutdownOutput() ` 方法，底层会告知服务端已经写完了，那么服务端收到消息后，就能知道已经读取完消息，如果服务端有要返回给客户的消息那么就可以通过服务端的输出流发送给客户端，如果没有，直接关闭 ` Socket ` 。 

**缺点：** 不能再次发送消息给服务端，如果再次发送，需要重新建立Socket连接

###  2.2.3 通过约定符号

这种方式的用法，就是双方约定一个字符或者一个短语，来当做消息发送完成的标识，通常这么做就需要改造读取方法。

假如约定单端的一行为END，代表发送完成，例如下面的消息，END则代表消息发送完成：

    
    
    hello world
    END
    

那么服务端响应的读取操作需要进行如下改造：

    
    
    Socket socket = server.accept();
    // 建立好连接后，从socket中获取输入流，并建立缓冲区进行读取
    BufferedReader read=new BufferedReader(new InputStreamReader(socket.getInputStream(),"UTF-8"));
    String line;
    StringBuilder stringBuilder = new StringBuilder();
    while ((line = read.readLine()) != null && !"END".equals(line)) {
      stringBuilder.append(line);
    }
    

  * **优点** ：不需要关闭流，当发送完一条命令（消息）后可以再次发送新的命令（消息） 
  * **缺点** ：需要额外的约定结束标志，太简单的容易出现在要发送的消息中，误被结束，太复杂的不好处理，还占带宽 

###  2.2.4 指定长度

约定一个定长头放在传输数据的最前端，用以标识数据体的长度。

客户端需要做的是，在发送消息之前先把消息的长度发送过去。

如果是需要服务器返回结果，那么也依然使用这种方式，服务端也是先发送结果的长度，然后客户端进行读取。

这样可以有效避免在发送方对数据进行分包发送时，误以为数据传输完成，因而接收不全的问题。

#  三、服务端优化

##  3.1 服务端并发处理能力

在实际开发中，一个Socket服务往往需要服务大量的Socket请求，那么就不能再服务完一个Socket的时候就关闭了，这时候可以采用循环接受请求并处理的逻辑，不过在实际生产中，很少用到这种方式，因为再处理一个Socket的同时会阻塞其他Socket。所以更多时候采用多线程的方式进行处理。

创建的线程会交给 **线程池** 来处理，优点：

  * 线程复用，创建线程耗时，回收线程慢 
  * 防止短时间内高并发，指定线程池大小，超过数量将等待，方式短时间创建大量线程导致资源耗尽，服务挂掉 

    
    
    package demo3;
    import java.io.InputStream;
    import java.net.ServerSocket;
    import java.net.Socket;
    import java.util.concurrent.ExecutorService;
    import java.util.concurrent.Executors;
    
    public class S {
        public static void main(String args[]) throws Exception {
            int port = 47639;
            ServerSocket server = new ServerSocket(port);
            System.out.println("server waits for connection...");
            //如果使用多线程，那就需要线程池，防止并发过高时创建过多线程耗尽资源
            ExecutorService threadPool = Executors.newFixedThreadPool(100);
            while (true) {
                Socket socket = server.accept();
                Runnable runnable=()->{
                    try {
                        InputStream inputStream = socket.getInputStream();
                        byte[] bytes = new byte[1024];
                        int len;
                        StringBuilder stringBuilder = new StringBuilder();
                        while ((len = inputStream.read(bytes)) != -1) {
                            stringBuilder.append(new String(bytes, 0, len, "UTF-8"));
                        }
                        System.out.println("get message from client: " + stringBuilder);
                        inputStream.close();
                        socket.close();
                    } catch (Exception e) {
                        e.printStackTrace();
                    }
                };
                threadPool.submit(runnable);
            }
    
        }
    }
    

使用线程池的方式，是一种成熟的方式，可以应用在生产工作中。

##  3.2 服务端其他属性

` ServerSocket ` 有以下3个属性:

  * ` SO_TIMEOUT ` ：表示等待客户连接的超时时间。一般不设置，会持续等待。 
  * ` SO_REUSEADDR ` ：表示是否允许重用服务器所绑定的地址，一般不设置。 
  * ` SO_RCVBUF ` ：表示接收数据的缓冲区的大小。一般不设置，用系统默认就可以了。 

##  3.3 NIO

性能不能满足需求的时候，就需要考虑使用NIO，本篇不展开。

#  四、其它知识

##  4.1 客户端绑定端口

服务端绑定端口是可以理解的，因为要监听指定的端口，但是客户端为什么要绑定端口，说实话我觉得这么做的人有点2，或许有的网络安全策略配置了端口访出，使用户只能使用指定的端口，那么这样的配置也是挺2的，直接说就可以不要留面子。

当然首先要理解的是，如果没有指定端口的话，Socket会自动选取一个可以用的端口，不用瞎操心的。

但是你非得指定一个端口也是可以的，做法如下，这时候就不能用Socket的构造方法了，要一步一步来：

    
    
    // 要连接的服务端IP地址和端口
    String host = "localhost"; 
    int port = 47639;
    // 与服务端建立连接
    Socket socket = new Socket();
    socket.bind(new InetSocketAddress(47639));
    socket.connect(new InetSocketAddress(host, port));
    

当这个程序执行完成以后，再次执行就会报，端口占用异常：

    
    
    java.net.BindException: Address already in use: connect
    

明明上一个Socket已经关闭了，为什么再次使用还会说已经被占用了呢？

用netstat 命令来查看端口的使用情况：

    
    
    netstat -n|findstr "47639"
    TCP 127.0.0.1:55534 127.0.0.1:47639 TIME_WAIT
    

发现端口的使用状态为 **TIME_WAIT** ，简单来说，当连接主动关闭后，端口状态变为 **TIME_WAIT**
，其他程序依然不能使用这个端口，防止服务端因为超时，重新发送的确认连接断开，对新连接的程序造成影响。

**TIME_WAIT** 的时间一般是2分钟、1分钟、30秒。

所以，客户端一般不要绑定端口

##  4.2 设置超时

    
    
    socket.connect(new InetSocketAddress(ipAddress, port), 1000);//设置连接请求超时时间1 s
    socket.setSoTimeout(30000);//设置读操作超时时间30 s
    

**timeout** \- 指定的以毫秒为单位的超时值。设置0为持续等待下去。建议根据网络环境和实际生产环境选择。

##  4.3 判断Socket是否可用

当需要判断一个Socket是否可用的时候，不能简简单单判断是否为null，是否关闭，下面给出一个比较全面的判断Socket是否可用的表达式，这是根据Socket自身的一些状态进行判断的，它的状态有：

  * bound：是否绑定 
  * closed：是否关闭 
  * connected：是否连接 
  * shutIn：是否关闭输入流 
  * shutOut：是否关闭输出流 

    
    
    socket != null && socket.isBound() && !socket.isClosed() && socket.isConnected()&& !socket.isInputShutdown() && !socket.isOutputShutdown()
    

这是第一步，保证Socket自身的状态是可用的，但是当连接正常创建后，上面的属性如果不调用本方相应的方法是不会改变的，也就是说如果网络断开、服务器主动断开，Java底层是不会检测到连接断开并改变Socket的状态。

所以，真实的检测连接状态还是得通过额外的手段，有两种方式。

###  4.3.1 自定义心跳包

跳包之所以叫心跳包是因为：它像心跳一样每隔固定时间发一次，以此来告诉服务器，这个客户端还活着。事实上这是为了保持长连接，至于这个包的内容，是没有什么特别规定的，不过一般都是很小的包，或者只包含包头的一个空包。

在TCP的机制里面，本身是存在有心跳包的机制的，也就是TCP的选项：SO_KEEPALIVE。系统默认是设置的2小时的心跳频率。但是它检查不到机器断电、网线拔出、防火墙这些断线。而且逻辑层处理断线可能也不是那么好处理。一般，如果只是用于保活还是可以的。

心跳包一般来说都是在逻辑层发送空的echo包来实现的。下一个定时器，在一定时间间隔下发送一个空包给客户端，然后客户端反馈一个同样的空包回来，服务器如果在一定时间内收不到客户端发送过来的反馈包，那就只有认定说掉线了。

其实，要判定掉线，只需要send或者recv一下，如果结果为零，则为掉线。但是，在长连接下，有可能很长一段时间都没有数据往来。理论上说，这个连接是一直保持连接的，但是实际情况中，如果中间节点出现什么故障是难以知道的。更要命的是，有的节点（防火墙）会自动把一定时间之内没有数据交互的连接给断掉。在这个时候，就需要我们的心跳包了，用于维持长连接，保活。

在获知了断线之后，服务器逻辑可能需要做一些事情，比如断线后的数据清理呀，重新连接呀……当然，这个自然是要由逻辑层根据需求去做了。

总的来说，心跳包主要也就是用于长连接的保活和断线处理。一般的应用下，判定时间在 **30-40秒** 。如果实在要求高，那就在 **6-9秒** 。

**心跳检测步骤：**

  * 客户端每隔一个时间间隔发生一个探测包给服务器 
  * 客户端发包时启动一个超时定时器 
  * 服务器端接收到检测包，应该回应一个包 
  * 如果客户机收到服务器的应答包，则说明服务器正常，删除超时定时器 
  * 如果客户端的超时定时器超时，依然没有收到应答包，则说明服务器挂了 

> 用 ` boolean socketFlag = socket.isConnected() && socket.isClosed() `
> 来判断是不靠谱的，这些方法都是访问 ` socket ` 在内存驻留的状态，当 ` socket ` 和服务器端建立链接后，即使 ` socket `
> 链接断掉了，调用上面的方法返回的仍然是链接时的状态，而不是 ` socket ` 的实时链接状态。
    
    
    private Thread thread;//循环发送心跳包的线程
    private Socket socket;//与服务器建立连接的socket对象
    private OutputStream outputStream;//输出流，用于发送心跳
    
    public void startThreadSocket() {
      try {
        if(!socket.getKeepAlive()) 
          socket.setKeepAlive(true);//true，若长时间没有连接则断开
        outputStream = socket.getOutputStream();//获得socket的输出流
        final String socketContent = "[这里为与服务器协商的特殊字符串，用于识别是发送的心跳信息]" + "\n";
        thread = new Thread(new Runnable() {
          @Override
          public void run() {
            while (true) {
              try {
                Thread.sleep(20 * 1000);//20s发送一次心跳
                outputStream.write(socketContent.getBytes("UTF-8"));
                outputStream.flush();
              } catch (Exception e) {
                e.printStackTrace();
              }
            }
          }
        });
        thread.start();
      } catch (Exception e) {
        e.printStackTrace();
      }
    }
    

###  4.3.2 通过发送紧急数据

` Socket ` 自带一种模式，那就是发送紧急数据，前提，服务端的 ` OOBINLINE ` 不能设置为 ` true ` ，它的默认值是 `
false ` 。

**影响**

  * 对客户端没有影响 
  * 对服务端，如果设置为 **true** ，那么服务端将会捕获紧急数据，这会对接收数据造成混淆，需要额外判断 

发送紧急数据通过调用 ` Socket ` 的方法：

    
    
    socket.sendUrgentData(0);
    

发送数据任意即可，因为 ` OOBINLINE ` 为 **false** 的时候，服务端会丢弃掉紧急数据。

当发送紧急数据报错以后，我们就会知道连接不通了。

###  4.3.3 是否需要判断连接断开

通过上面的两种方式已经可以判断出连接是否可用，然后我们就可以进行后续操作，可是请大家认真考虑下面的问题：

  1. 发送心跳成功时确认连接可用，当再次发送消息时能保证连接还可用吗？即便中间的间隔很短 
  2. 如果连接不可用了，重新建立连接再次发送数据？还是单单记录日志？ 
  3. 如果打算重新建立连接，为何不在发送异常时再新建连接？ 

**看如下案例：**

提前检测连接是否可用：

    
    
    //有一个连接中的socket
    Socket socket=...
    //要发送的数据
    String data="";
    try{
        //发送心跳包或者紧急数据，来检测连接的可用性
    }catch (Excetption e){
        //打印日志，并重连Socket
        socket=new Socket(host,port);
    }
    socket.write(data);
    

直接发送数据，出异常后重新连接再次发送：

    
    
    //有一个连接中的socket
    Socket socket=...
    //要发送的数据
    String data="";
    try{
        socket.write(data);
    }catch (Excetption e){
        //打印日志，并重连Socket
        socket=new Socket(host,port);
        socket.write(data);
    }
    

  * 两种方式均可实现连接断开重新连接并发送 
  * 提前检测，再每次发送消息的时候都要检测，影响效率，占用带宽 
  * 是否必要自行选择 

##  4.4 设置端口重用

首先，创建Socket时， ` SO_REUSEADDR ` 默认是禁止的，设置 **true** 有什么作用呢，Java API中是这么介绍的：

> 关闭 TCP 连接时，该连接可能在关闭后的一段时间内保持超时状态（通常称为 TIME_WAIT 状态或 2MSL
> 等待状态）。对于使用已知套接字地址或端口的应用程序而言，如果存在处于超时状态的连接（包括地址和端口），可能不能将Socket绑定到所需的
> SocketAddress 上。

使用 ` bind(SocketAddress) ` 绑定套接字前启用 ` SO_REUSEADDR ` 允许在上一个连接处于超时状态时绑定套接字。

一般是用在绑定端口的时候使用，但是经过我的测试建议如下：

  * 服务端绑定端口后，关闭服务端，重新启动后不会提示端口占用 
  * 客户端绑定端口后，关闭，即便设置 ` ReuseAddress ` 为 **true** ，即便能绑定端口，连接的时候还是会报端口占用异常 

综上所述，不建议绑定端口，也没必要设置 ` ReuseAddress ` 。

##  4.5 设置关闭等待

启用/禁用具有指定逗留时间（以秒为单位）的 **SO_LINGER** 。最大超时值是特定于平台的。

该设置仅影响套接字关闭。

当调用Socket的close方法后，没有发送的数据将不再发送，设置这个值的话，Socket会等待指定的时间发送完数据包。

  * 对于一般数据量来说，即便直接关闭Socket的连接，服务端也是可以收到数据的。 

  * 对于一般应用没必要设置这个值，当数据量发送过大抛出异常时，再来设置这个值也不晚。 

  * 超时后，套接字将通过 TCP RST 强制性关闭。 

  * **启用超时值为0** 的选项将立即强制关闭。如果指定的超时值大于 65535，则其将被减少到 65535。 

##  4.6 设置发送延迟策略

**延迟策略 TCP_NODELAY**

一般来说当客户端想服务器发送数据的时候，会根据当前数据量来决定是否发送，如果数据量过小，那么系统将会根据Nagle
算法，来决定发送包的合并，也就是说发送会有延迟。

比如说对实时性要求很高的消息发送，电子竞技游戏等，即便数据量很小也要求立即发送，如果稍有延迟就会感觉到卡顿，默认情况下Nagle
算法是开启的，所以如果不打算有延迟，最好关闭它。这样一旦有数据将会立即发送而不会写入缓冲区。

但是对延迟要求不是特别高下，还是可以提升网络传输效率的。

##  4.7 设置输出输出缓冲区大小

  * SO_SNDBUF：发送缓冲 
  * SO_RCVBUF：接收缓冲 

默认都是8K，如果有需要可以修改，通过相应的set方法。需要注意如下：

  * 设置太小数据传输将过于频繁。 

  * 太大了将会造成消息停留。 

##  4.8 设置保持连接存活

> 对于构建长时间连接的Socket，应该配置上SO_KEEPALIVE。

虽然说当设置连接连接的读超时为0，即无限等待时，Socket不会被主动关闭。

但是总会有莫名其妙的软件来检测你的连接是否有数据发送，长时间没有数据传输的连接会被它们关闭掉。

**设置SO_KEEPALIVE为true** ，当一段时间内在任意方向上都没有跨越socket交换数据，则 TCP 会自动发送一个保持存活的消息到对面。

将会有以下三种响应：

  * 返回期望的 **ACK** 。那么不通知应用程序（因为一切正常），2 小时的不活动时间过后，TCP 将发送另一个探头。 

  * 对面返回 **RST** ，表明对面挂了，但是又好了，Socket依然要关闭 

  * 没有响应，说明对面挂了，这时候关闭Socket 

##  4.10 拆包和黏包

> 大部分情况下我们是不希望发生拆包和黏包的

  * 拆包：当一次发送（Socket）的数据量过大，而底层（TCP/IP）不支持一次发送那么大的数据量，则会发生拆包现象。 
  * 黏包：当在短时间内发送（Socket）很多数据量小的包时，底层（TCP/IP）会根据一定的算法（指Nagle）把一些包合作为一个包发送。 

###  4.10.1 黏包

黏包实际上是对网络通信的一种优化，假如说上层只发送一个字节数据，而底层却发送了41个字节，其中20字节的I P首部、 20字节的T C
P首部和1个字节的数据，而且发送完后还需要确认，这么做浪费了带宽，量大时还会造成网络拥堵。

当然它还是有一定的缺点的，就是因为它会合并一些包会导致数据不能立即发送出去，会造成延迟，如果能接受（一般延迟为200ms），那么还是不建议关闭这种优化。

如果不希望发生黏包，那么通过禁用TCP_NODELAY即可，Socket中也有相应的方法：

    
    
    socket.setTcpNoDelay(true) 
    

通过设置为true即可防止在发送的时候黏包，但是当 **发送的速率大于读取的速率时** ，在 **服务端也会发生黏包**
，服务端读取过慢，导致一次可能读取多个包。

###  4.10.2 拆包

如何应对拆包，其实在上面2.3节已经介绍过了，那就是如何表明发送完一条消息了，对于已知数据长度的模式，可以构造相同大小的数组，循环读取，示例代码如下：

    
    
    int length=1024;//这个是读取的到数据长度，现假定1024
    byte[] data=new byte[1024];
    int readLength=0;
    while(readLength<length){
        int read = inputStream.read(data, readLength, length-readLength);
        readLength+=read;
    }
    

