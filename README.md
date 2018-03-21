### Load Balancer High-Level Design

The solution provides the address(IP) of the least loaded node among the available nodes, based on load conditions & other attributes. 
This load balancer solution will expose a VIP (Virtual-IP) to the client applications. 
Within load balancer, there will be one MASTER LB physical node and multiple Slave LB nodes(can be extended depending on the load).

### Message exchange between Master node and Slave nodes
### Master LB Node
Master node and slave nodes are heart-beating with each other. 
They do so by periodically exchanging HELLO & HELLO-ACK messages which are essentially 0 data payload. 
Other than heartbeat messages, the Master node also receives the updates(PUSH_LOAD_INFO) from all Client/Slaves nodes. 
These messages carry actual load information of Client nodes. 
Master node will maintain a hash table where the key will be IP address of the Client LB nodes and value will be their load.

### Slave LB Node
Client LB nodes periodically send load updates(via PUSH_LOAD_INFO) of their gateways to the Master LB node. 
Apart from PUSH_LOAD_INFO, Slave LB nodes keep sending hello messages for liveness assurance to the Master LB node.

The ###communication protocol### between Master LB node and Slave LB nodes Master and Slaves will use TCP protocol to communicate with each other which will ensure guaranteed delivery ion packets.

### Data Structures

enum lb_node_type

{

  LB_NODE_MASTER;
  
  LB_NODE_CLIENT;
  
  LB_NODE_MAX;
  
};

struct lb


{
  lb_node_type    node_type;	/* Master or Slave*/
  
  union node {
  
    lb_master master;	/* Data specific to lb Maser */
    
    lb_slave slave;    /* Data specific to lb Slave */
	  
  }
  
lb_timer timer;		/* Timer used depending on Master or Slave */

lb_socket_context sock_ctx;    /* node socket address */

lb_load_info load_info;	/* node load info*/

}

### Master LB

enum lb_master_state

{

  lb_MASTER_ST_UP_LISTENING;
  
  lb_MASTER_ST_DOWN;
  
}

struct lb_master

{

  lb_master_state state;
  
  lb_slave *lb_slave_table[MAX_LB_SLAVES]; /* connected slaves */
  
}

### Slave LB

enum lb_slave_state

{ 

  LB_SLAVE_CONNECTED;
  
  LB_SLAVE_PUSH_LOAD_INFO;	/* Slave push */
  
  LB_SLAVE_DOWN; 
  
}

struct lb_slave

{

  lb_slave_state state;
  
  lb_msg *msg; /* node message receiving or writing depending on state */
  
}

### LB messages(LB header + LB payload)

enum lb_msg_type

{ 

  MSG_PUSH_LOAD_INFO;
  
  MSG_HELLO;
  
  MSG_HELLO_ACK;
  
}lb_message;

### Header

#define lb_MSG_HDR_SIZE 4 /*bytes*/

typedef struct lb_msg_hdr_t

{

  uint16 payload_type;   /* should match lb_MSG_XXXX */
  
  uint16 payload_len;     /* payload len; excludes header lb_MSG_HDR_SIZE */
  
}lb_msg_hdr;

### Header + Payload

typedef struct lb_msg_t

{

  lb_msg_hdr header;
  
  char payload[MAX_SIZE];
  
  uint total_processed_len; 	 /* Read or Write */
  
  boolean doneProcessing;      /* done processing*/
  
}lb_msg;
