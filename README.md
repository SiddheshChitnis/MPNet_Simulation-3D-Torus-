MPNet_Simulation-3D-Torus-
==========================
1)First File- Initialization

import peersim.core.*;
import peersim.config.Configuration;

public class KVSInit implements Control
{
private static final String PAR_PROT = "protocol";
	private static final String PAR_NUMSERVER = "numServer";
	/** protocol id */
	private int pid = 0;
	
	/** number of servers */
	private int numServer;
	public KVSInit(String prefix)
	{
		pid = Configuration.getPid(prefix + "." + PAR_PROT);
		numServer = Configuration.getInt(prefix + "." + PAR_NUMSERVER);
	}
	
	/**
	 * the method to run when running this initialization class
	 */
	public boolean execute()
	{
		System.out.println("The number of server is:" + numServer);
		Library.initLib(numServer);
		Library.dimSize = Library.getDimSize(numServer);
		System.out.println("The dimension is:" + Library.dimSize);
		for (int i = 0;i < Network.size();i++)
		{
			Node node = (Node)Network.get(i);
			
			/** get the protocol for a node */
			PeerProtocol pp = (PeerProtocol)node.getProtocol(pid);
			
			/** set the node id to be consecutive integers */
			pp.id = i;
			
			pp.getCoordinate();
		}
		System.out.println("The number of server is:" + numServer);
		Library.initLib(numServer);
		Library.dimSize = Library.getDimSize(numServer);
		System.out.println("The dimension is:" + Library.dimSize);
		return false;
	}
}

2) Second FIle - The Message

public class Message
{
	public int originSenderid;
	public int currentSenderid;
	public int nextSenderid;
	public String data;
	public double datasize;
	public int terminationid;
	
	public Message(int osId, int cSid, int nSid, String data, double ds, int tID)
	{
		this.originSenderid = osId;
		this.currentSenderid = cSid;
		this.nextSenderid = nSid;
		this.data = data;
		this.datasize = ds;
		this.terminationid = tID;
	}
}
	
	3) Library File
	
	import peersim.core.Network;
import peersim.core.Node;
import peersim.core.CommonState;
import peersim.edsim.EDSimulator;

import java.io.*;

public class Library 
{

	public static long netSpeed;
	public static long latency;
	public static BufferedWriter bw;
	public static long numOperaFinished;
	public static long numAllOpera;
	public static boolean allowParallel;
	
	public static long numMsg;
	public static int dimSize;
	
	public static int getDimSize(int num)
	{
		return (int)Math.cbrt((double)(num));
	}
	
	public static void initLib(int numServer)
	{
		Library.netSpeed = 10L * 1024L * 1024L * 1024L;
		Library.latency = 100L;
		
	}	
	public static long commOverhead(long msgSize)
	{
		return msgSize / Library.netSpeed + Library.latency;
	}
	
	public static int[] onetoThreeMapping(int index, int dimSize) throws ArithmeticException
	{
		int[] coordinate = new int[3];
	//	System.out.println("Test"+dimSize);
		coordinate[0] = index / (dimSize * dimSize); 
		coordinate[1] = (index - coordinate[0] * (dimSize * dimSize)) / dimSize;
		coordinate[2] = index % dimSize;
		return coordinate;
	}
	
	public static int threetoOneMapping(int[] coordinate, int dimSize)
	{
		return coordinate[0] * dimSize * dimSize + coordinate[1] * dimSize + coordinate[2];
	}
}

4) PeerProtocol - The main part

import peersim.config.Configuration;
import peersim.core.CommonState;
import peersim.core.Network;
import peersim.core.Node;
import peersim.edsim.EDProtocol;
import peersim.edsim.EDSimulator;

import java.util.*;
import java.io.*;

public class PeerProtocol implements EDProtocol
{
	private static final String PARA_TRANPORT = "transport";
	private static final String PARA_NUMSERVER = "numServer";
	
	private Parameters par;
	private int numServer;
	
	public int id;

	public String prefix;
	
	public int[] coordinate;
	public int[] neighbor;
	
	public PeerProtocol(String prefix)
	{
		this.prefix = prefix;
		par = new Parameters();
		par.tid = Configuration.getPid(prefix + "." + PARA_TRANPORT);

		numServer = Configuration.getInt(prefix + "." + PARA_NUMSERVER);
		
	}
	
	public int[]  path(int[] source,int[] desti)
	 {
		
		 
				 if (Arrays.equals(source, desti))
				 {    
					 System.out.println(" \n\n**************DESTINATION REACHED*********** ");
				 }
				 
					
					int diff1 = desti[0]-source[0];
					int diff2 = desti[1]-source[1];
					int diff3 = desti[2]-source[2];
					
					if (diff1>0)
					{
					   
					for ( int p = 0 ; p<diff1 ; p++)
					{
						source[0]=source[0]+1;
						break;
								//System.out.print("---->"+source[0]+""+source[1]+""+source[2]);
					}
					}
					else if(diff1<0)
					{   
						for ( int p = 0 ; p<Math.abs(diff1) ; p++)
						{
							source[0]=source[0]-1;
							break;
									//System.out.print("---->"+source[0]+""+source[1]+""+source[2]);
						}
					}
						
					
				
					
					else if(diff2>0)
					{
					   
					for ( int p = 0 ; p<diff2 ; p++)
					{
						source[1]=source[1]+1;
						break;
								//System.out.print("---->"+source[0]+""+source[1]+""+source[2]);
					}
					}
					else if((diff2<0))
					{
						   
						for ( int p = 0 ; p<Math.abs(diff2) ; p++)
						{
							source[1]=source[1]-1;
							break;
									//System.out.print("---->"+source[0]+""+source[1]+""+source[2]);
						}
					}
				
					else if (diff3>0)
					{
					   
					for ( int p = 0 ; p<diff3 ; p++)
					{
						source[2]=source[2]+1;
						break;
								//System.out.print("---->"+source[0]+""+source[1]+""+source[2]);
					}
					}
					else if(diff3<0)
					{
						   
						for ( int p = 0 ; p<Math.abs(diff3) ; p++)
						{
							source[2]=source[2]-1;
							break;
									//System.out.print("---->"+source[0]+""+source[1]+""+source[2]);
						}
					}
			
			
					//System.out.println(" \n\n**************DESTINATION REACHED*********** ");
			return source;					     	
		
	 }
	
	public void getCoordinate()
	{
		coordinate = Library.onetoThreeMapping(id, Library.dimSize);
	}
	
	public void processEvent(Node node, int pid, Object event)
	{
		par.pid = pid;
		Message msg = (Message)event;
		int dest = msg.terminationid;
		Library.numMsg++;
		if (dest == id)
		{
			
			System.out.println("The throughput is:" + ((double)Library.numMsg / 
					CommonState.getTime()));
			System.out.println("The no of links used is :"+Library.numMsg);
			int edge;
			edge = Network.size() + 4; 
			System.out.println("Total no of links available is:"+edge);
			return;
		}
		
		int[] destCoordiate = Library.onetoThreeMapping(dest, Library.dimSize);
		//System.out.println("**"+coordinate[0]+"**"+coordinate[1]+"**"+coordinate[2]);
		int[] inter = path(coordinate, destCoordiate);
		int interIndex = Library.threetoOneMapping(inter, Library.dimSize);
		Message msgNext = new Message(msg.originSenderid, id, 
		interIndex, msg.data, msg.datasize, msg.terminationid);
		//System.out.println("**"+Library.commOverhead((long)msg.datasize)+"**");
		long time = CommonState.getTime() + Library.commOverhead((long)msg.datasize);
		//System.out.println("$$"+CommonState.getTime()+"$$");
		Node nodeNext = (Node)Network.get(interIndex);
		EDSimulator.add(time, msgNext, nodeNext, pid);
	}
	
	public Object clone()
	{
		PeerProtocol pp = new PeerProtocol(prefix);
		//pp.hm = new HashMap<String, String>();
		return pp;
	}
}

5)Relating the neighbouring nodes

import java.lang.Math;
import java.util.Scanner;
public class Neighbours {
	 public static void main(String[] args) {
		 
		 create();
		 
		 
	 }
		 
		 
		 public static void  create() 
		 {	 
	
	int i = 0; 
	int j=0 ;
	int k=0 ;
	//int n=0;
	
	  
	System.out.println("Enter the number of nodes");
	
	Scanner scanner=new Scanner(System.in);
	int m=scanner.nextInt();
	
	double n=nodes(m)-1;
	System.out.println("The returned value is"+n);
	
	for ( i=0 ;i<=n;i++)
	{
		//System.out.println(i);
		
		for (j=0 ; j<=n ;  j++)
		{ 
			
			  
			  for(k=0 ;k<=n ; k++)
			  {
				
				  
				  System.out.println("The Node is : "+i+","+j+","+k);
				
				 
				
				 Object[] neigh =  check(i,j,k,n);
				 int[] neigh1 = (int[])neigh[0];
					int[] neigh2 =(int[])neigh[1];
					int[] neigh3 = (int[])neigh[2];
					int p = (int)neigh[3];
					for(int q=0;q<p;q++)
				 {
				 	System.out.println(" "+neigh1[q]+""+neigh2[q]+""+neigh3[q]);
				 	

				 	 
				 }
                 
			  }
		}
		
		
		
	}
	
	}
	 
	 
	 public static double nodes(int m)
	 {
		 //int n=0;
		 return Math.cbrt(m);
	 }


	public static Object[] check(int a, int b, int c, double limit){
		
		Object[] neigh = new Object[4];
		int[] neigh1 = new int[6];
		int[] neigh2 =new int[6];
		int[] neigh3 = new int[6];
		int p=0;
		System.out.println("The Neighbours are ");


	
		if (a>0){
			neigh1[p]=a-1;
			neigh2[p]=b;
			neigh3[p]=c;
			//System.out.println(" "+};
			p++;
	}
		if (a<limit){
			neigh1[p]=a+1;
			neigh2[p]=b;
			neigh3[p]=c;
			//System.out.println(" "+};
			p++;
	}

		if (b>0){
			neigh1[p]=a;
			neigh2[p]=b-1;
			neigh3[p]=c;
			//System.out.println(" "+};
			p++;
	}
		if (b<limit){
			neigh1[p]=a;
			neigh2[p]=b+1;
			neigh3[p]=c;
			//System.out.println(" "+};
			p++;
	}
		
		if (c>0){
			neigh1[p]=a;
			neigh2[p]=b;
			neigh3[p]=c-1;
			//System.out.println(" "+};
			p++;
	}
		if (c<limit){
			neigh1[p]=a;
			neigh2[p]=b;
			neigh3[p]=c+1;
			//System.out.println(" "+};
			p++;
	}
	
		
		
		
neigh[0]=neigh1;
neigh[1]=neigh2;
neigh[2]=neigh3;
neigh[3]=p;


return neigh;


}
	
	}
	
6)Traffic Generation

import peersim.core.*;
import peersim.config.Configuration;
import peersim.edsim.EDSimulator;

import java.util.*;
import java.io.*;

public class TrafficGene implements Control
{
	private static final String PAR_PROT = "protocol";
	private static final String PAR_NUMSERVER = "numServer";
	
	private final int pid;
	private final int numServer;
	
	public TrafficGene(String prefix)
	{
		pid = Configuration.getPid(prefix + "." + PAR_PROT);
		numServer = Configuration.getInt(prefix + "." + PAR_NUMSERVER, 1);
	}
	 
	public boolean execute()
	{
		int size = Network.size();
	//	for(int i=0;i<1000;i++)
	//	{
		Message msg = new Message(0, 0, -1, "Hello, "
				+ "I am sending you a message", 100,9999);
		Node node = (Node)Network.get(0);
		PeerProtocol pp = (PeerProtocol)node.getProtocol(pid);
		int[] tmp = Library.onetoThreeMapping(Network.size() - 1, Library.dimSize);
		int[] inter = pp.path(pp.coordinate, tmp);
		int interId = Library.threetoOneMapping(inter, Library.dimSize);
		msg.nextSenderid = interId;
		long time = Library.commOverhead((long)msg.datasize);
		Node destNode = (Node)Network.get(interId);
		EDSimulator.add(time, msg, destNode, pid);
	//	}
		return false;
	}
}

7) Configuration File (needs to be changed for ever experiment)
simulation.endtime 10^16
simulation.logtime 10^6

simulation.experiments 1

network.size 10000

protocol.tr UniformRandomTransport
{
	mindelay 9
	maxdelay 10

}

protocol.node PeerProtocol
{
	transport tr
	numServer 10000
}

init.create KVSInit
{
	protocol node
	numServer 10000
}

control.workloadgene TrafficGene
{
	protocol node
	numServer 10000
	step simulation.endtime
}

