/**
 * 地铁线路图查询器（学生版）
 * 编译：gcc -o metro metro_student.c -std=c99
 * 运行：.\metro.exe
 */
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <ctype.h>
#include <limits.h>

#define MAX_NAME_LEN 32
#define MAX_LINE_NAME 20

// 边结点
typedef struct EdgeNode {
    int adjVex;
    int weight;
    struct EdgeNode *next;
} EdgeNode;

// 顶点（站点）
typedef struct VertexNode {
    char name[MAX_NAME_LEN];
    EdgeNode *firstEdge;
    int *lineIds;
    int lineCount;
} VertexNode;

// 图
typedef struct Graph {
    VertexNode *vertices;
    int vertexNum;
    int vertexCapacity;
    int edgeNum;
    int isDirected;
} Graph;

// BFS队列
typedef struct Queue {
    int *data;
    int front, rear, size, capacity;
} Queue;

// 函数声明
Graph* createGraph(int initCapacity, int isDirected);
void resizeGraph(Graph *g);
int addVertex(Graph *g, const char *name);
int findVertexIndex(Graph *g, const char *name);
void addEdge(Graph *g, int u, int v, int weight);
void addLineToStation(Graph *g, int stationIdx, int lineId);
void readMetroFile(const char *filename, Graph *g);
void printAdjList(Graph *g);

void DFSRecursive(Graph *g, int v, int *visited);
void DFSTraversal(Graph *g, int start);
void BFSTraversal(Graph *g, int start);
void connectivityAnalysis(Graph *g);
void dijkstra(Graph *g, int start, int *dist, int *prev);
void printPath(Graph *g, int *prev, int start, int end);
void shortestPathByTime(Graph *g, int start, int end);
void shortestPathByTransfer(Graph *g, int start, int end);
void freeGraph(Graph *g);

void printMenu();
Queue* createQueue(int capacity);
void enqueue(Queue *q, int val);
int dequeue(Queue *q);
int isEmpty(Queue *q);
void freeQueue(Queue *q);

// 主函数
int main() {
    Graph *g = createGraph(100, 0);
    readMetroFile("metro.txt", g);

    int choice, start, end;
    char startName[MAX_NAME_LEN], endName[MAX_NAME_LEN];
    do {
        printMenu();
        printf("请输入选择：");
        scanf("%d", &choice);
        getchar();

        switch (choice) {
            case 1:
                printAdjList(g);
                break;
            case 2:
                printf("请输入起始站点名称：");
                fgets(startName, MAX_NAME_LEN, stdin);
                startName[strcspn(startName, "\n")] = '\0';
                start = findVertexIndex(g, startName);
                if (start == -1) {
                    fprintf(stderr, "错误：站点 '%s' 不存在。\n", startName);
                } else {
                    printf("\nDFS 遍历序列（从 %s 开始）：\n", startName);
                    DFSTraversal(g, start);
                }
                break;
            case 3:
                printf("请输入起始站点名称：");
                fgets(startName, MAX_NAME_LEN, stdin);
                startName[strcspn(startName, "\n")] = '\0';
                start = findVertexIndex(g, startName);
                if (start == -1) {
                    fprintf(stderr, "错误：站点 '%s' 不存在。\n", startName);
                } else {
                    printf("\nBFS 遍历序列（从 %s 开始）：\n", startName);
                    BFSTraversal(g, start);
                }
                break;
            case 4:
                connectivityAnalysis(g);
                break;
            case 5:
                printf("请输入起点站：");
                fgets(startName, MAX_NAME_LEN, stdin);
                startName[strcspn(startName, "\n")] = '\0';
                printf("请输入终点站：");
                fgets(endName, MAX_NAME_LEN, stdin);
                endName[strcspn(endName, "\n")] = '\0';
                start = findVertexIndex(g, startName);
                end = findVertexIndex(g, endName);
                if (start == -1) fprintf(stderr, "错误：起点 '%s' 不存在。\n", startName);
                else if (end == -1) fprintf(stderr, "错误：终点 '%s' 不存在。\n", endName);
                else shortestPathByTime(g, start, end);
                break;
            case 6:
                printf("请输入起点站：");
                fgets(startName, MAX_NAME_LEN, stdin);
                startName[strcspn(startName, "\n")] = '\0';
                printf("请输入终点站：");
                fgets(endName, MAX_NAME_LEN, stdin);
                endName[strcspn(endName, "\n")] = '\0';
                start = findVertexIndex(g, startName);
                end = findVertexIndex(g, endName);
                if (start == -1) fprintf(stderr, "错误：起点 '%s' 不存在。\n", startName);
                else if (end == -1) fprintf(stderr, "错误：终点 '%s' 不存在。\n", endName);
                else shortestPathByTransfer(g, start, end);
                break;
            case 0:
                printf("退出程序。\n");
                break;
            default:
                printf("无效选择，请重新输入。\n");
        }
        printf("\n");
    } while (choice != 0);

    freeGraph(g);
    return 0;
}

// 基础图操作
Graph* createGraph(int initCapacity, int isDirected) {
    Graph *g = (Graph*)malloc(sizeof(Graph));
    g->vertexCapacity = initCapacity;
    g->vertexNum = 0;
    g->edgeNum = 0;
    g->isDirected = isDirected;
    g->vertices = (VertexNode*)malloc(sizeof(VertexNode) * initCapacity);
    for (int i = 0; i < initCapacity; i++) {
        g->vertices[i].name[0] = '\0';
        g->vertices[i].firstEdge = NULL;
        g->vertices[i].lineIds = NULL;
        g->vertices[i].lineCount = 0;
    }
    return g;
}

void resizeGraph(Graph *g) {
    int newCap = g->vertexCapacity * 2;
    g->vertices = (VertexNode*)realloc(g->vertices, sizeof(VertexNode) * newCap);
    for (int i = g->vertexCapacity; i < newCap; i++) {
        g->vertices[i].name[0] = '\0';
        g->vertices[i].firstEdge = NULL;
        g->vertices[i].lineIds = NULL;
        g->vertices[i].lineCount = 0;
    }
    g->vertexCapacity = newCap;
}

int addVertex(Graph *g, const char *name) {
    int idx = findVertexIndex(g, name);
    if (idx != -1) return idx;
    if (g->vertexNum >= g->vertexCapacity) resizeGraph(g);
    strcpy(g->vertices[g->vertexNum].name, name);
    g->vertices[g->vertexNum].firstEdge = NULL;
    g->vertices[g->vertexNum].lineIds = NULL;
    g->vertices[g->vertexNum].lineCount = 0;
    return g->vertexNum++;
}

int findVertexIndex(Graph *g, const char *name) {
    for (int i = 0; i < g->vertexNum; i++) {
        if (strcmp(g->vertices[i].name, name) == 0)
            return i;
    }
    return -1;
}

void addEdge(Graph *g, int u, int v, int weight) {
    if (u < 0 || u >= g->vertexNum || v < 0 || v >= g->vertexNum) return;
    EdgeNode *e1 = (EdgeNode*)malloc(sizeof(EdgeNode));
    e1->adjVex = v;
    e1->weight = weight;
    e1->next = g->vertices[u].firstEdge;
    g->vertices[u].firstEdge = e1;
    g->edgeNum++;
    if (!g->isDirected) {
        EdgeNode *e2 = (EdgeNode*)malloc(sizeof(EdgeNode));
        e2->adjVex = u;
        e2->weight = weight;
        e2->next = g->vertices[v].firstEdge;
        g->vertices[v].firstEdge = e2;
    }
}

void addLineToStation(Graph *g, int stationIdx, int lineId) {
    if (stationIdx < 0 || stationIdx >= g->vertexNum) return;
    VertexNode *v = &g->vertices[stationIdx];
    for (int i = 0; i < v->lineCount; i++)
        if (v->lineIds[i] == lineId) return;
    v->lineCount++;
    v->lineIds = (int*)realloc(v->lineIds, sizeof(int) * v->lineCount);
    v->lineIds[v->lineCount - 1] = lineId;
}

// 全新读取函数，只用fscanf，不会出现0站点
void readMetroFile(const char *filename, Graph *g) {
    FILE *fp = fopen(filename, "r");
    if (!fp) {
        fprintf(stderr, "无法打开文件 %s\n", filename);
        exit(1);
    }
    int routeCount;
    fscanf(fp, "%d", &routeCount);
    int dummy;
    fscanf(fp, "%d", &dummy);

    for (int rid = 0; rid < routeCount; rid++) {
        int lineId, stationCnt;
        fscanf(fp, "%d %d", &lineId, &stationCnt);
        int prev = -1;
        int time = 1;
        int readCnt = 0;
        while (readCnt < stationCnt) {
            int numVal;
            int ret = fscanf(fp, "%d", &numVal);
            if (ret == 1) {
                time = numVal;
                continue;
            }
            char buf[64];
            fseek(fp, -1L, SEEK_CUR);
            fscanf(fp, "%s", buf);
            int cur = addVertex(g, buf);
            addLineToStation(g, cur, lineId);
            if (prev != -1) {
                addEdge(g, prev, cur, time);
                time = 1;
            }
            prev = cur;
            readCnt++;
        }
    }
    fclose(fp);
    printf("成功读取地铁数据：共 %d 个站点，%d 条边。\n", g->vertexNum, g->edgeNum);
}

void printAdjList(Graph *g) {
    printf("\n===== 邻接表 =====\n");
    for (int i = 0; i < g->vertexNum; i++) {
        printf("%s (%d条线路): ", g->vertices[i].name, g->vertices[i].lineCount);
        EdgeNode *p = g->vertices[i].firstEdge;
        while (p) {
            printf("-> %s(%dmin) ", g->vertices[p->adjVex].name, p->weight);
            p = p->next;
        }
        printf("\n");
    }
    printf("\n===== 换乘站 =====\n");
    for (int i = 0; i < g->vertexNum; i++) {
        if (g->vertices[i].lineCount > 1)
            printf("%s:%d 条线路\n", g->vertices[i].name, g->vertices[i].lineCount);
    }
}

void printMenu() {
    printf("\n====== 地铁查询系统 ======\n");
    printf("1. 输出邻接表和换乘站\n");
    printf("2. DFS 遍历（从指定站点）\n");
    printf("3. BFS 遍历（从指定站点）\n");
    printf("4. 连通分量分析\n");
    printf("5. 最短路径（最少时间）\n");
    printf("6. 最短路径（最少换乘）\n");
    printf("0. 退出\n");
}

// 队列实现
Queue* createQueue(int capacity) {
    Queue *q = (Queue*)malloc(sizeof(Queue));
    q->capacity = capacity;
    q->data = (int*)malloc(sizeof(int) * capacity);
    q->front = q->rear = q->size = 0;
    return q;
}
void enqueue(Queue *q, int val) {
    if (q->size >= q->capacity) return;
    q->data[q->rear] = val;
    q->rear = (q->rear + 1) % q->capacity;
    q->size++;
}
int dequeue(Queue *q) {
    if (isEmpty(q)) return -1;
    int ret = q->data[q->front];
    q->front = (q->front + 1) % q->capacity;
    q->size--;
    return ret;
}
int isEmpty(Queue *q) { return q->size == 0; }
void freeQueue(Queue *q) { free(q->data); free(q); }

// DFS
void DFSRecursive(Graph *g, int v, int *visited) {
    if (v < 0 || v >= g->vertexNum || visited[v]) return;
    visited[v] = 1;
    printf("%s ", g->vertices[v].name);
    EdgeNode *p = g->vertices[v].firstEdge;
    while (p != NULL) {
        if (!visited[p->adjVex])
            DFSRecursive(g, p->adjVex, visited);
        p = p->next;
    }
}
void DFSTraversal(Graph *g, int start) {
    if (g == NULL || start < 0 || start >= g->vertexNum) return;
    int *visited = (int*)calloc(g->vertexNum, sizeof(int));
    DFSRecursive(g, start, visited);
    printf("\n");
    free(visited);
}

// BFS
void BFSTraversal(Graph *g, int start) {
    if (g == NULL || start < 0 || start >= g->vertexNum) return;
    int *visited = (int*)calloc(g->vertexNum, sizeof(int));
    Queue *q = createQueue(g->vertexNum);
    visited[start] = 1;
    enqueue(q, start);
    printf("%s ", g->vertices[start].name);
    while (!isEmpty(q)) {
        int v = dequeue(q);
        EdgeNode *p = g->vertices[v].firstEdge;
        while (p != NULL) {
            if (!visited[p->adjVex]) {
                visited[p->adjVex] = 1;
                printf("%s ", g->vertices[p->adjVex].name);
                enqueue(q, p->adjVex);
            }
            p = p->next;
        }
    }
    printf("\n");
    freeQueue(q);
    free(visited);
}

// 连通分量
void connectivityAnalysis(Graph *g) {
    if (g == NULL) return;
    int *visited = (int*)calloc(g->vertexNum, sizeof(int));
    int componentCount = 0;
    printf("\n===== 连通分量分析 =====\n");
    for (int i = 0; i < g->vertexNum; i++) {
        if (!visited[i]) {
            componentCount++;
            printf("连通分量 %d: ", componentCount);
            Queue *q = createQueue(g->vertexNum);
            visited[i] = 1;
            enqueue(q, i);
            printf("%s ", g->vertices[i].name);
            while (!isEmpty(q)) {
                int v = dequeue(q);
                EdgeNode *p = g->vertices[v].firstEdge;
                while (p != NULL) {
                    if (!visited[p->adjVex]) {
                        visited[p->adjVex] = 1;
                        printf("%s ", g->vertices[p->adjVex].name);
                        enqueue(q, p->adjVex);
                    }
                    p = p->next;
                }
            }
            printf("\n");
            freeQueue(q);
        }
    }
    printf("总连通分量数：%d\n", componentCount);
    free(visited);
}

// Dijkstra
void dijkstra(Graph *g, int start, int *dist, int *prev) {
    if (g == NULL || start < 0 || start >= g->vertexNum) return;
    int vNum = g->vertexNum;
    int *visited = (int*)calloc(vNum, sizeof(int));
    for (int i = 0; i < vNum; i++) {
        dist[i] = INT_MAX;
        prev[i] = -1;
        visited[i] = 0;
    }
    dist[start] = 0;
    for (int i = 0; i < vNum; i++) {
        int u = -1;
        int minD = INT_MAX;
        for (int j = 0; j < vNum; j++) {
            if (!visited[j] && dist[j] < minD) {
                minD = dist[j];
                u = j;
            }
        }
        if (u == -1) break;
        visited[u] = 1;
        EdgeNode *p = g->vertices[u].firstEdge;
        while (p != NULL) {
            int v = p->adjVex;
            if (!visited[v] && dist[u] != INT_MAX && dist[u] + p->weight < dist[v]) {
                dist[v] = dist[u] + p->weight;
                prev[v] = u;
            }
            p = p->next;
        }
    }
    free(visited);
}

void printPath(Graph *g, int *prev, int start, int end) {
    if (start == end) {
        printf("%s", g->vertices[start].name);
        return;
    }
    if (prev[end] == -1) {
        printf("无法到达");
        return;
    }
    printPath(g, prev, start, prev[end]);
    printf(" -> %s", g->vertices[end].name);
}

// 最少时间路径
void shortestPathByTime(Graph *g, int start, int end) {
    if (g == NULL || start < 0 || end < 0 || start >= g->vertexNum || end >= g->vertexNum) {
        printf("无效站点编号\n");
        return;
    }
    if (start == end) {
        printf("起点终点相同，无需出行\n");
        return;
    }
    int vNum = g->vertexNum;
    int *dist = (int*)malloc(sizeof(int) * vNum);
    int *prev = (int*)malloc(sizeof(int) * vNum);
    dijkstra(g, start, dist, prev);
    printf("\n最短时间路径：");
    if (dist[end] == INT_MAX) printf("两点无通路");
    else {
        printPath(g, prev, start, end);
        printf("，总耗时 %d 分钟", dist[end]);
    }
    printf("\n");
    free(dist);
    free(prev);
}

// 最少换乘路径
void shortestPathByTransfer(Graph *g, int start, int end) {
    if (g == NULL || start < 0 || end < 0 || start >= g->vertexNum || end >= g->vertexNum) {
        printf("无效站点编号\n");
        return;
    }
    if (start == end) {
        printf("起点终点相同，无需出行\n");
        return;
    }
    int vNum = g->vertexNum;
    typedef struct { int u, v, weight; } EdgeInfo;
    EdgeInfo *edgeInfos = (EdgeInfo*)malloc(sizeof(EdgeInfo) * g->edgeNum);
    int edgeCount = 0;
    for (int u = 0; u < vNum; u++) {
        EdgeNode *p = g->vertices[u].firstEdge;
        while (p != NULL) {
            if (u < p->adjVex) {
                edgeInfos[edgeCount].u = u;
                edgeInfos[edgeCount].v = p->adjVex;
                edgeInfos[edgeCount].weight = p->weight;
                edgeCount++;
            }
            p->weight = 1;
            p = p->next;
        }
    }
    int *dist = (int*)malloc(sizeof(int) * vNum);
    int *prev = (int*)malloc(sizeof(int) * vNum);
    dijkstra(g, start, dist, prev);
    printf("\n最少换乘路径：");
    if (dist[end] == INT_MAX) printf("两点无通路");
    else {
        printPath(g, prev, start, end);
        printf("，换乘 %d 次", dist[end] - 1);
    }
    printf("\n");
    // 恢复权值
    for (int i = 0; i < edgeCount; i++) {
        int u = edgeInfos[i].u, v = edgeInfos[i].v, w = edgeInfos[i].weight;
        EdgeNode *p = g->vertices[u].firstEdge;
        while (p && p->adjVex != v) p = p->next;
        if (p) p->weight = w;
        p = g->vertices[v].firstEdge;
        while (p && p->adjVex != u) p = p->next;
        if (p) p->weight = w;
    }
    free(dist);
    free(prev);
    free(edgeInfos);
}

// 释放整张图
void freeGraph(Graph *g) {
    if (!g) return;
    for (int i = 0; i < g->vertexNum; i++) {
        EdgeNode *p = g->vertices[i].firstEdge;
        while (p != NULL) {
            EdgeNode *tmp = p;
            p = p->next;
            free(tmp);
        }
        free(g->vertices[i].lineIds);
    }
    free(g->vertices);
    free(g);
}