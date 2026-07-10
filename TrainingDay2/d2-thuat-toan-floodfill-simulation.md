# Thuật toán Flood Fill

> Thuật toán tìm đường được sử dụng phổ biến trong Micromouse.
---

# Khái quát thuật toán Flood Fill

Flood Fill là thuật toán giúp chuột xác định hướng đi đến đích bằng cách gán cho mỗi ô trong mê cung một giá trị thể hiện **khoảng cách từ ô đó đến đích**.

Bắt đầu từ ô đích với giá trị **0**, thuật toán lan dần ra các ô xung quanh, mỗi ô nhận giá trị lớn hơn ô trước đó **1 đơn vị**.

Sau khi toàn bộ mê cung được gán giá trị, chuột chỉ cần di chuyển sang ô lân cận có **giá trị nhỏ nhất** để tiến gần hơn đến đích.

---


# Code thuật toán

## Bước 1. Reset khoảng cách

* Gán toàn bộ `dist = INF`.

---

## Bước 2. Khởi tạo đích

* Gán các ô đích bằng `0`.
* Đưa tất cả các ô đích vào hàng đợi (`queue`).

---

## Bước 3. BFS lan từ đích

Lần lượt lấy từng ô trong `queue`.

**Điều kiện lan sang ô mới (4 ô kề cạnh)**

* Không bị tường chặn.
* Nằm trong bản đồ.

**Ví dụ**

* Từ ô `(0,0)` sang `(0,1)` theo hướng **Đông**.
* Cần kiểm tra xem phía **Đông** của ô `(0,0)` có tường hay không.

BFS đóng vai trò như là một thuật toán giúp tìm đường đi ngắn nhất từ đích đến các ô trên mê cung và ngược lại.
#### Tài liệu tham khảo thuật toán: https://wiki.vnoi.info/algo/graph-theory/breadth-first-search.md

---

## Bước 4. Cập nhật khoảng cách

Nếu có thể đi sang ô mới:

```cpp
if(dist[new] > dist[current] + 1){
    dist[new] = dist[current] + 1;
    q.push(new);
}
```

---

## Bước 5. Lặp đến khi BFS hoàn tất

Kết quả:

Mỗi ô trong mê cung sẽ chứa **khoảng cách ngắn nhất đến đích**.

---

## Bước 6. Robot di chuyển

Tại mỗi bước:

* Xét 4 ô xung quanh.
* Chọn ô có `dist` nhỏ nhất.
* Di chuyển sang ô đó.
* Cập nhật vị trí mouse.

---

## Bước 7. Lặp lại

Sau mỗi lần di chuyển:

* Cập nhật thông tin tường mỗi khi đi đến ô mới(`wall[x][y][direct]`).
* Chạy lại Flood Fill.
* Tiếp tục cho đến khi đến đích.

---

## Bước 8. Di chuyển lại ô bắt đầu

* Gọi lại thuật toán với đích là ô `(0,0)`.

---

# Simulation Micromouse

## Cài đặt

* https://github.com/mackorone/mms#download
* https://github.com/mackorone/mms/releases

---

## Bước 1

Thêm hai file:

* `API.h`
* `API.cpp`

vào thư mục chứa code Flood Fill.

---

## Bước 2

Giải nén file `.zip`, mở thư mục và chạy:

```text
mms.exe
```


### Giao diện Simulation

![image](https://hackmd.io/_uploads/ByUwY3czGe.png)

---

## Bước 3

Nhấn nút kế bên thanh mouse:

![image](https://hackmd.io/_uploads/Sk_W535Mzl.png)



### Giao diện Build & Run

![image](https://hackmd.io/_uploads/BJf0q35zzg.png)

---

## Bước 4

Thiết lập:

* File `.cpp`
* Thư mục chứa source
* Build Command
* Run Command

---

## Bước 5

Nhấn **Build & Run** để chạy simulator.

---
# Video giả lập thuật toán

LINK: [**VIDEO GIẢ LẬP THUẬT TOÁN**](https://drive.google.com/file/d/1WgLowixIZCv5VtUHNwRQdTUT0oVqCcj9/view)

# Code tham khảo
:::spoiler code
```cpp
#include <iostream>
#include <queue>
#include <cstdint>
#include <climits>
#include <cstring>
#include <vector>
#include "API.h" //simulator
#define pr pair <int, int>
#define se second
#define fi first
using namespace std;

const int N = 16; // kích cỡ mê cung
const int inf = 255;

enum Direct {
    NORTH = 0,
    EAST = 1,
    SOUTH = 2,
    WEST = 3
};

// Hướng di chuyển 
int dx[4] = {0, 1, 0, -1};
int dy[4] = {1, 0, -1, 0};

int dist[N][N]; //lưu khoảng cách sau khi chạy thuật toán
bool wall[N][N][4]; //lưu tường


int cur_x = 0, cur_y = 0, cur_dir = NORTH; // robot xuất phát ở vị trí (0; 0) nhìn hướng Bắc
pair <int, int> cen[4] = {{7, 7}, {7, 8}, {8, 7}, {8, 8}}; // các ô trung tâm


bool inMap(int x, int y) //Kiểm tra ô có trong map không
{
    return x>=0 && x<N && y>=0 && y<N;
}

bool isGoal(int x, int y) // kiểm tra có đến đích chưa
{
    return (x == 7 || x == 8) && (y == 7 || y == 8);
}

void set_wall(int x, int y, int direct) //set có tương hay không
{
    wall[x][y][direct] = true;

    int u = x + dx[direct];
    int v = y + dy[direct];

    if (inMap(u, v))
        wall[u][v][(direct + 2) % 4] = 1; //set cho ô kề cạnh
    
    // Vẽ tường lên simulator - dùng mảng
    const char dir_chars[4] = {'n', 'e', 's', 'w'};
    if (direct >= 0 && direct < 4) {
        API::setWall(x, y, dir_chars[direct]);
    }
}

void update_wall()
{
    bool f, l, r;

    f = API::wallFront();
    l = API::wallLeft();
    r = API::wallRight();

    if(f) set_wall(cur_x, cur_y, cur_dir);
    if(l) set_wall(cur_x, cur_y, (cur_dir + 3) % 4);
    if(r) set_wall(cur_x, cur_y, (cur_dir + 1) % 4);
    

   
}

void flood_fill()
{
    // reset dist
    for(int i=0;i<N;i++)
        for(int j=0;j<N;j++)
            dist[i][j] = inf;

    queue <pr> q;

    for (auto i : cen) // thêm ô trung tâm vào queue
    {
        dist[i.fi][i.se] = 0;
        q.push(i);
    }

    while (!q.empty()) //bfs
    {
        int u = q.front().fi;
        int v = q.front().se;
        q.pop();

        for (int i = 0; i < 4; i++)
        {
            if (wall[u][v][i]) continue;

            int nx = u + dx[i];
            int ny = v + dy[i];

            if (!inMap(nx, ny)) continue;

            if (dist[nx][ny] > dist[u][v] + 1)
            {
                dist[nx][ny] = dist[u][v] + 1;
                q.push({nx, ny});
            }
        }
    }
}

int chooseDir() // chọn hướng đi ngắn nhất
{
    int best = inf;
    int bestDir = -1;

    for(int i = 0; i < 4; i++)
    {
        if(wall[cur_x][cur_y][i]) continue;

        int nx = cur_x + dx[i];
        int ny = cur_y + dy[i];

        if(!inMap(nx,ny)) continue;

        if(dist[nx][ny] < best)
        {
            best = dist[nx][ny];
            bestDir = i;
        }
    }

    return bestDir;
}

void move(int targetD) 
{
    if(targetD == -1) return;
    
    int diff = (targetD - cur_dir + 4) % 4; // hướng di chuyển dựa trên hướng hiện tại

    if (diff == 1)
    {
        API::turnRight();
    }
    else if (diff == 3) 
    {
        API::turnLeft();
    }
    else if (diff == 2)
    {
        API::turnLeft();
        API::turnLeft();
    }

    API::moveForward();

    cur_dir = targetD;
    cur_x += dx[cur_dir];
    cur_y += dy[cur_dir];
}

void show_path() // simulation-hiện thị đường đi ngắn nhất hiện tại
{
    int x = cur_x, y = cur_y;
    API::clearAllColor();

    while(!isGoal(x,y))
    {
        API::setColor(x, y, 'G');

        int best = inf;
        int nx = x, ny = y;

        for(int d=0; d<4; d++)
        {
            if(wall[x][y][d]) continue;

            int tx = x + dx[d];
            int ty = y + dy[d];

            if(!inMap(tx,ty)) continue;

            if(dist[tx][ty] < best)
            {
                best = dist[tx][ty];
                nx = tx;
                ny = ty;
            }
        }

        if(nx == x && ny == y) break;

        x = nx;
        y = ny;
    }

    // tô goal
    for(auto i : cen)
        API::setColor(i.fi, i.se, 'R');
}


int main()
{
    memset(wall, 0, sizeof(wall));

    flood_fill();
    
    
    for(;;)
    {
        update_wall();
        flood_fill();

        for(int i = 0; i < N; i++)
        {
            for(int j = 0; j < N; j++)
            {
                if(dist[i][j] < inf)
                    API::setText(i, j, to_string(dist[i][j]));
                else
                    API::setText(i, j, "X");
            }
        }
    
        show_path(); //simulation
        
        if (isGoal(cur_x, cur_y))
        {
                break;
        }
        
        int best_dir = chooseDir();
        if(best_dir == -1) { // lỗi, không tìm ra đường đi
            break;
        }
        
        move(best_dir);
        API::setText(cur_x, cur_y, to_string(dist[cur_x][cur_y]));
        
    }
    
    cerr << "Mission complete!" << endl;
    return 0;
}
```

***Lưu ý***
* Code tham khảo chỉ phù hợp cho simulation (test code) trước khi nạp vào mouse. Các hàm API phải được thay bằng các hàm tương ứng.
---
# Video tham khảo
**LINK:** https://www.youtube.com/watch?v=Zwh-QNlsurI
[![Nhấn vào để xem video Dynamic Programming / Flood Fill Algorithm](https://img.youtube.com/vi/Zwh-QNlsurI/0.jpg)](https://www.youtube.com/watch?v=Zwh-QNlsurI)
