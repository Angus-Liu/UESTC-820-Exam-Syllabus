# PV 操作大题

1. N 个生产者进程和 M 个消费者进程共享大小为 K 的缓冲区，遵循规则如下：
（1）进程之间必须以互斥方式访问缓冲区；
（2）对每一条放入缓冲区的数据，所有消费者都必须接收 1 次；
（3）缓冲区满时，生产者必须阻塞；
（4）缓冲区空时，消费者必须阻塞。
请用 P、V 操作实现其同步过程，须说明信号量含义。
    
    ```c
    // TODO: 待修改，不是最优解
    semaphore mutex = 1;       // 用于缓冲区的互斥访问
    semaphore empty[M] = {K};  // 用于表示每个消费者对应空缓冲区的信号量
    semaphore full[M] = {0};   // 用于表示每个消费者对应满缓冲区的信号量
    
    producer() {
    	while(true) {
    		for(int i = 0; i < M; i++) {
    			P(empty[i]);
    			P(mutex);
    			// 放入数据
    			V(mutex);
    		  V(full[i]);
    		}
    	}
    }
    
    consumer(int i) {
    	while(true) {
    		P(full[i]);
    		P(mutex);
    		// 取出数据
    		V(mutex);
    		V(empty[i]);
    	}
    }
    ```
    
    ---
    
2. 有桥如下图所示。车流方向如箭头所示。回答如下问题：
    - ① 假设桥上每次只能有一辆车行驶，试用信号灯的 P，V 操作实现交通管理。
    - ② 假设桥上不允许两车交会，但允许同方向多辆车一次通过（即桥上可有多辆同方向 行驶的车）。试用信号灯的 P，V 操作实现桥上的交通管理。
    
    ![Untitled](PV%20%E6%93%8D%E4%BD%9C%E5%A4%A7%E9%A2%98%208c57ccf8b0cb4546b2f22a8305f3d08e/Untitled.png)
    
    解答：
    
    - ①  桥上每次只能有一辆车行驶，所以只要设置一个信号量 bridge 就可判断桥是否使用，若在使用中，等待；若无人使用，则执行 P 操作进入；出桥后，执行 V 操作。
        
        ```c
        semaphore bridge = 1; // 用于互斥地访问桥
        
        NtoS() { // 从北向南
            P(bridge);
            通过桥;
            V(bridge);
        }
        
        StoN() { // 从南向北
            P(bridge);
            通过桥;
            V(bridge);
        }
        ```
        
    - ②  桥上可以同方向多车行驶，需要设置 bridge，还需要对同方向车辆计数。为了防止同方向计数过程中同时申请 bridge 造成同方向不可同时行车的问题，要对计数过程加以保护，因此设置信号量 mutexSN 和 mutexNS。
        
        ```c
        semaphore bridge = 1;  // 用于互斥地访问桥
        int countNS = 0;       // 用于表示从北到南的汽车数量
        semaphore mutexNS = 1; // 用于互斥访问 countNS 变量
        int countSN = 0;       // 用于表示从南到北的汽车数量
        semaphore mutexSN = 1; // 用于互斥访问 countSN 变量
        
        NtoS() { // 从北向南
        		P(mutexNS);
        		if(countNS == 0)
                P(bridge);
        		countNS++;
        		V(mutexNS);
            
            通过桥;
            
            P(mutexNS);
        		countNS--;
        		if(countNS == 0)
                V(bridge);
        		V(mutexNS);
        }
        
        StoN() { // 从南向北
        		P(mutexSN);
        		if(countSN == 0)
                P(bridge);
        		countSN++;
        		V(mutexSN);
            
            通过桥;
            
            P(mutexSN);
        		countSN--;
        		if(countSN == 0)
                V(bridge);
        		V(mutexSN);
        }
        ```