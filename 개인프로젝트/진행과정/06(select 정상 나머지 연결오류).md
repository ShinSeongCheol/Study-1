# 6안

Client
```
package CMP;

import java.io.*;
import java.net.*;
import java.util.*;

public class Client {
    public static void main(String[] args) {
    	try {
    		String serverIp = "127.0.0.1";
    		System.out.println("서버와 연결중..");
    		
    		//소켓 생성
    		Socket socket = new Socket(serverIp, 5001);
    		// 해당 주소의 5001포트로 tcp연결 요청을 보냄.
    		BufferedReader br = null;
    		BufferedWriter bw = null;
    		// 문장 형태의 문자열을 주고받을수 있도록 셋팅.
    		br = new BufferedReader(new InputStreamReader(socket.getInputStream()));
    		bw = new BufferedWriter(new OutputStreamWriter(socket.getOutputStream()));
    		// client는 연결이 성공하면 서버가 보낸 메시지를 모니터에 출력.
    		System.out.println(br.readLine());
    		System.out.println(br.readLine());
    		// 그담부터 사용자로부터 계속해서 메시지를 입력 받을때마다 서버에 전달.
    		while (true) {
    			//키보드 값 저장
    			Scanner scan = new Scanner(System.in);
    			String msg = scan.nextLine();
			//조건에 따라 키보드값 서버로 전송
    			switch(msg) {
				case "select":
					bw.write(msg + "\n");
					bw.flush(); 
					break;
				case "insert":
					String[] i = new String[15];
					System.out.println("이름 생년월일 전화번호 메모 :");
					String i1 = scan.next();
					String i2 = scan.next();
					String i3 = scan.next();
					String i4 = scan.next();
					i[0] = "'"+i1+"'" +", "+"'"+i2+"'"+", "+"'" +i3+ "'" +", "+"'"+i4+"'";
					//System.out.println(i[0]); 
					bw.write(msg +"\n");
					bw.flush();
					bw.write(i[0]+ "\n");
					bw.flush();
					break;
				case "delete":
					String[] d = new String[5];
					System.out.println("삭제하려는 카테고리 :");
					String d1 = scan.next();
					System.out.println("삭제하려는 내용 :");
					String d2 = scan.next();
					d[0] = d1 + " = " + "'" + d2 + "'";
					bw.write(msg+"\n");
					bw.flush();
					bw.write(d[0]+"\n");
					bw.flush();
					break;
				case "update":
					String[] u = new String[11];
					System.out.println("이동할 조건행 :");
					String u1 = scan.next();
					System.out.println("검색할 내용 :");
					String u2 = scan.next();
					System.out.println("수정할 내용 :");
					String u3 = scan.next();
					u[0] = u1 + " = " + "'" + u3 + "'" + " WHERE " + u1 + " = " + "'" + u2 + "'";
					bw.write(msg+"\n");
					bw.flush();
					bw.write(u[0]+"\n");
					bw.flush();
					break;
				default:
					bw.write(msg + "\n");
					bw.flush(); 
					break;
			}
    			
    			System.out.println(br.readLine());
    			int q;
    			int w = Integer.parseInt(br.readLine());
    			switch(w) {
    				case 1:
    					System.out.println(br.readLine());
    					break;
    				default :
    					for(q = 1; q <= w ; q++) {
    	    				System.out.println(br.readLine());
    	    			}
    					break;
    			}
    		}
    		
    	} catch (UnknownHostException e) {
    		e.printStackTrace();
    	}catch (IOException e) {
    		e.printStackTrace();
    	}
    }
}
```
Server
```
package CMP;

import java.io.*;
import java.net.*;
import java.util.Date;
import java.text.SimpleDateFormat;

public class Server {
    public static void main(String[] args) {
        //서버소켓 정의
        ServerSocket serverSocket = null;
        //받아올 소켓 정의
        Socket socket = null;
        //읽어올 br 정의
        BufferedReader br = null;
        //쓸 bw 정의
        BufferedWriter bw = null;
        //DB 클래스 객체화
        MariaDB test = new MariaDB();
        try {
            // server는 프로그램이 실행되면 연결요청을 기다림 포트 5001
            serverSocket = new ServerSocket(5001);
            socket = serverSocket.accept();
            // 문장 형태의 문자열을 주고받을수 있도록 셋팅.
            br = new BufferedReader(new InputStreamReader(socket.getInputStream()));
            bw = new BufferedWriter(new OutputStreamWriter(socket.getOutputStream()));
            // server는 client가 연결되면 '명령어를 입력하세요 : ' 라고 메시지 전송.
            bw.write("연결되었습니다.\n");
            bw.flush();
            bw.write("명령어를 입력하세요 : \n");
            bw.flush();

            // 계속해서 client가 보내는 메시지를 수신,
            while(true) {
                String msg = br.readLine();
                switch(msg) {
                    //조회 select * from tcp;
                    case "select":
                        //DB 클래스 호출
                        test.select();
                        // 클라이언트에게 전송
                        bw.write(test.a);
                        bw.flush();
                        bw.write(Integer.toString(test.i)/*★*/+"\n");
                        bw.flush();
                        //System.out.println(test.i);
                        if(test.b1 == "1") {
                            for(int j=0;j<test.i;j++) {
                                bw.write(test.b[j] + "\n");
                                bw.flush();
                            }
                        }else if(test.b1 == "2") {
                            bw.write("Database 연결중 에러가 발생 했습니다." + "\n");
                            bw.flush();
                        }
                        break;
                    //삽입 
                    case "insert":
                        //DB 클래스 호출
                        String i1 = br.readLine();
                        test.insert(i1);
                        bw.write(test.a);
                        bw.flush();
                        bw.write(Integer.toString(test.i)+"\n");
                        bw.flush();
                        if(test.c1 == "1") {
                            bw.write( "데이터 삽입 성공"+ "\n");
                            bw.flush();
                        }else if(test.c1 == "2") {
                            bw.write("데이터 삽입 실패" + "\n");
                            bw.flush();
                        }
                        break;
                    //삭제 
                    case "delete":
                        //DB 클래스 호출
                        String d1 = br.readLine();
                        test.delete(d1);
                        bw.write(test.a);
                        bw.flush();
                        bw.write(Integer.toString(test.i)+"\n");
                        bw.flush();
                        if(test.d1 == "1") {
                            bw.write( "데이터 삭제 성공"+ "\n");
                            bw.flush();
                        }else if(test.d1 == "2") {
                            bw.write("데이터 삭제 실패" + "\n");
                            bw.flush();
                        }
                        break;
                    //수정 
                    case "update":
                        //DB 클래스 호출
                        String u1 = br.readLine();
                        test.update(u1);
                        bw.write(test.a);
                        bw.flush();
                        bw.write(Integer.toString(test.i)+"\n");
                        bw.flush();
                        if(test.e1 == "1") {
                            bw.write("데이터 수정 성공" + "\n");
                            bw.flush();
                        }else if(test.e1 == "2") {
                            bw.write("데이터 수정 실패" + "\n");
                            bw.flush();
                        }

                        break;
                    default:
                        bw.write("그런 명령어는 없습니다. 아래 명령 중 골라주세요."+"\n");
                        bw.flush();
                        bw.write(Integer.toString(1)+"\n");
                        bw.flush();
                        bw.write("select, insert, delete, update"+"\n");
                        bw.flush();
                        break;
                }
                System.out.println(getTime() +" "+ msg);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    static String getTime() {
        SimpleDateFormat f = new SimpleDateFormat("[MM월 dd일 hh:mm]");
        return f.format(new Date());
    }
}
```
MariaDB
```
package CMP;
import java.sql.*;

public class MariaDB {
	private static final String DB_DRIVER_CLASS = "org.mariadb.jdbc.Driver";
	private static final String DB_URL = "jdbc:mariadb://127.0.0.1:3306/";
	String a, b1, c, c1, d, d1, e, e1;
	String[] b;
	Connection conn = null;
	Statement stmt = null;
	ResultSet rs = null;
	public int i;
	public String connectDB() {
		try {
			Class.forName(DB_DRIVER_CLASS);
			conn = DriverManager.getConnection(DB_URL, "root", "password");
			//"연결성공"			
			a = "DB 연결 성공 \n";
			i = 1;
		} catch (ClassNotFoundException e) {
			//"드라이브 로딩 실패"
			a = "드라이브 로딩 실패 \n";
			i = 1;
		} catch (SQLException e) {
			//"DB 연결 실패"
			a = "DB 연결 실패 \n";
			i = 1;
		}
		return a;
	}
	public String[] select(){
		try {
			//connectDB 호출
			connectDB();
			//배열 크기 설정;
			// 혹시 오류나면 다시 옆에거로 b = new String[100];
			b = new String[7];
			// SQL문을 데이터베이스에 보내기위한 stmt 객체 생성
			stmt = conn.createStatement();
			// SQL실행
			rs = stmt.executeQuery(" use tcp; ");
			rs = stmt.executeQuery(" select * from tcp; ");
			// 결과값 출력
			b1 = "1";
			i= 0;
			while (rs.next()) {
				String 나이 = rs.getString(1);
				String 생년월일 = rs.getString(2);
				String 전화번호 = rs.getString(3);
				String 메모 = rs.getString(4);
				b[i] = 나이 + " " + 생년월일 + " " + 전화번호 + " " + 메모;
				//System.out.println(b[i]);
				i++;
			}
			//System.out.println(i);
		}catch (SQLException e) {
			//Database 연결중 에러가 발생 했습니다.
			b1 = "2";
			i = 1;
		}
		return b;
	}
	public void insert(String ii) {
		//insert 명령어
		try {
			
			c1 = "1";
			connectDB();
			stmt = conn.createStatement();
			rs = stmt.executeQuery(" use tcp; ");
			rs = stmt.executeQuery(" insert into tcp values(" +ii+ ");");
			i = 1;
			
		}catch(SQLException e) {
			//연결 에러
			c1 = "2";
			i = 1;
		}
	}
	public void delete(String dd) {
		//delete 명령어
		try {
			d1 = "1";
			connectDB();
			stmt = conn.createStatement();
			rs = stmt.executeQuery(" use tcp; ");
			rs = stmt.executeQuery("delete from tcp where "+dd+";");
			i = 1;
		}catch(SQLException e) {
			d1 = "2";
			i = 1;
		}
	}
	public void update(String uu) {
		//update 명령어
		try {
			e1 = "1";
			connectDB();
			stmt = conn.createStatement();
			rs = stmt.executeQuery(" use tcp; ");
			rs = stmt.executeQuery("UPDATE tcp set "+uu+";");
			i = 1;
		}catch(SQLException e) {
			e1 = "2";
			i = 1;
		}
	}
}
```