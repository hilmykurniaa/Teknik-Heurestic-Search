# Teknik Heurestic Search
# 1. Pelajari class EightPuzzleSearch, EightPuzzleSpace, dan Node.
- EightPuzzleSearch adalah kelas yang digunakan dalam konteks permainan teka-teki delapan kotak untuk menerapkan algoritma yang mencari solusi. Kelas ini bertanggung jawab untuk mengatur pencarian dan mengevaluasi kemungkinan langkah-langkah menuju solusi.
- EightPuzzleSpace adalah kelas yang menggambarkan ruang keadaan dalam permainan teka-teki delapan kotak. Kelas ini mencakup informasi tentang keadaan saat ini, langkah-langkah yang telah diambil, dan aturan permainan. EightPuzzleSpace membantu menjaga integritas dan representasi dari setiap keadaan yang dieksplorasi selama pencarian solusi.
- Node adalah objek yang dimanfaatkan dalam algoritma pencarian. Node ini mewakili satu keadaan dalam ruang pencarian dan menyimpan informasi mengenai keadaan itu sendiri, biaya atau jarak yang diperlukan untuk mencapainya, serta informasi penting lainnya. Node membantu dalam melacak dan memahami keadaan selama proses pencarian solusi dan memungkinkan algoritma untuk memutuskan langkah apa yang harus diambil selanjutnya.

# 2. Ubahlah initial dan goal state dari program di atas sehingga bentuk initial dan goal statenya Gambar 5.8. Kemudian tentukan langkah-langkah mana saja sehingga puzzlenya mencapai goal state. Analisa dan bedakan dengan solusi pada point 1.
# Code:
    package JavaApplivation2;
    import java.util.Vector;

    class Node {
    int[] state = new int[9];
    int cost;
    Node parent = null;
    Vector<Node> successors = new Vector<Node>();

    Node(int s[], Node parent) {
        this.parent = parent;
        for (int i = 0; i < 9; i++)
            state[i] = s[i];
    }

    public Node() {
    }

    public String toString() {
        String s = "";
        for (int i = 0; i < 9; i++) {
            s = s + state[i] + " ";
        }
        return s;
    }

    public boolean equals(Object node) {
        Node n = (Node) node;
        boolean result = true;
        for (int i = 0; i < 9; i++) {
            if (n.state[i] != state[i])
                result = false;
        }
        return result;
    }

    Vector<Node> getPath(Vector<Node> v) {
        v.insertElementAt(this, 0);
        if (parent != null)
            v = parent.getPath(v);
        return v;
    }

    Vector<Node> getPath() {
        return getPath(new Vector<Node>());
    }

    public static void main(String args[]) {
        new Node().run();
    }

    private void run() {
    }
    }

    class EightPuzzleSpace {
    Node getRoot() {
        int initialState[] = {3, 1, 2, 4, 7, 5, 6, 8, 0};
        return new Node(initialState, null);
    }

    Node getGoal() {
        int goalState[] = {1, 2, 3, 4, 0, 8, 5, 6, 7};
        return new Node(goalState, null);
    }

    Vector<Node> getSuccessors(Node parent) {
        Vector<Node> successors = new Vector<Node>();
        for (int r = 0; r < 3; r++) {
            for (int c = 0; c < 3; c++) {
                if (parent.state[(r * 3) + c] == 0) {
                    if (r > 0) {
                        successors.add(transformState(r - 1, c, r, c, parent));
                    }
                    if (r < 2) {
                        successors.add(transformState(r + 1, c, r, c, parent));
                    }
                    if (c > 0) {
                        successors.add(transformState(r, c - 1, r, c, parent));
                    }
                    if (c < 2) {
                        successors.add(transformState(r, c + 1, r, c, parent));
                    }
                }
            }
        }
        parent.successors = successors;
        return successors;
    }

    Node transformState(int r0, int c0, int r1, int c1, Node parent) {
        int[] s = parent.state;
        int[] newState = { s[0], s[1], s[2], s[3], s[4], s[5], s[6], s[7], s[8] };
        newState[(r1 * 3) + c1] = s[(r0 * 3) + c0];
        newState[(r0 * 3) + c0] = 0;
        return new Node(newState, parent);
    }

    public static void main(String args[]) {
        new EightPuzzleSpace().run();
    }

    private void run() {
    }
    }

    class EightPuzzleSearch {
    EightPuzzleSpace space = new EightPuzzleSpace();
    Vector<Node> open = new Vector<Node>();
    Vector<Node> closed = new Vector<Node>();

    int h1Cost(Node node) {
        int cost = 0;
        for (int i = 0; i < node.state.length; i++) {
            if (node.state[i] != i) cost++;
        }
        return cost;
    }

    int h2Cost(Node node) {
        int cost = 0;
        int state[] = node.state;
        for (int i = 0; i < state.length; i++) {
            int v0 = i, v1 = state[i];
            if (v1 == 0)
                continue;
            int row0 = v0 / 3, col0 = v0 % 3, row1 = v1 / 3, col1 = v1 % 3;
            int c = (Math.abs(row0 - row1) + Math.abs(col0 - col1));
            cost += c;
        }
        return cost;
    }

    /* Boleh diubah dengan memakai heuristic h1 atau h2 */
    int hCost(Node node) {
        return h1Cost(node);
    }

    Node getBestNode(Vector nodes) {
        int index = 0, minCost = Integer.MAX_VALUE;
        for (int i = 0; i < nodes.size(); i++) {
            Node node = (Node) nodes.elementAt(i);
            if (node.cost < minCost) {
                minCost = node.cost;
                index = i;
            }
        }
        Node bestNode = (Node) nodes.remove(index);
        return bestNode;
    }

    int getPreviousCost(Node node) {
        int i = open.indexOf(node);
        int cost = Integer.MAX_VALUE;
        if (i != -1) {
            cost = open.get(i).cost;
        } else {
            i = closed.indexOf(node);
            if (i != -1)
                cost = closed.get(i).cost;
        }
        return cost;
    }

    void printPath(Vector path) {
        for (int i = 0; i < path.size(); i++) {
            System.out.print(" " + path.elementAt(i) + "\n");
        }
    }

    void run() {
        Node root = space.getRoot();
        Node goal = space.getGoal();
        Node solution = null;
        open.add(root);
        System.out.print("\nRoot: " + root + "\n\n");
        while (open.size() > 0) {
            Node node = getBestNode(open);
            int pathLength = node.getPath().size();
            closed.add(node);
            if (node.equals(goal)) {
                solution = node;
                break;
            }
            Vector<Node> successors = space.getSuccessors(node);
            for (int i = 0; i < successors.size(); i++) {
                Node successor = successors.get(i);
                int cost = hCost(successor) + pathLength + 1;
                int previousCost;
                previousCost = getPreviousCost(successor);
                boolean inClosed;
                inClosed = closed.contains(successor);
                boolean inOpen = open.contains(successor);
                if (!(inClosed || inOpen) || cost < previousCost) {
                    if (inClosed)
                        closed.remove(successor);
                    if (!inOpen)
                        open.add(successor);
                    successor.cost = cost;
                    successor.parent = node;
                }
            }
        }
        if (solution != null) {
            Vector path = solution.getPath();
            System.out.print("\nSolution found\n");
            printPath(path);
        }
    }

    public static void main(String args[]) {
        new EightPuzzleSearch().run();
    }
    }

Dalam kode Eight Puzzle yang telah diberikan, langkah-langkah untuk mencapai keadaan tujuan dari keadaan awal dapat ditemukan dalam class `EightPuzzleSearch`. Class ini bertanggung jawab untuk mencari solusi dengan menggunakan algoritma pencarian A* dan memungkinkan pengguna untuk memilih heuristic yang akan digunakan baik itu `h1` atau `h2`. Proses pencarian solusi melibatkan beberapa tahap berikut:

1. Pertama inisialisasikan keadaan awal dan keadaan tujuan.
2. Kemudian buat node awal (root) menggunakan keadaan awal tersebut.
3. Masukkan node awal ke dalam daftar `open`.
4. Selanjutnya, selama daftar `open` belum kosong lakukan langkah-langkah berikut:
   a. Pilih node dengan biaya terendah, yang dihitung berdasarkan heuristic dan panjang path.
   b. Tambahkan node terpilih ke dalam daftar `closed` untuk menghindari pengulangan.
   c. Jika node terpilih sama dengan keadaan tujuan, maka solusi telah ditemukan, dan proses berakhir.
   d. Selanjutnya, dapatkan semua node penerus dari node terpilih.
   e. Untuk setiap node penerus, hitung biaya baru berdasarkan heuristic, panjang path, dan biaya sebelumnya.
   f. Jika node penerus belum pernah ada di daftar `open` atau memiliki biaya lebih rendah, tambahkan node penerus ke daftar `open` dengan biaya yang telah diperbarui.

Tahapan ini akan terus diulang sampai solusi ditemukan atau semua kemungkinan solusi dieksplorasi. Hasil akhir dari pencarian akan mencetak langkah-langkah solusi ke dalam output.
Penting untuk dicatat bahwa class `EightPuzzleSearch`, `EightPuzzleSpace`, dan `Node` bekerja sama untuk mencari dan mengeksekusi solusi Eight Puzzle. Class `EightPuzzleSearch` mengatur logika pencarian, `EightPuzzleSpace` memberikan informasi tentang keadaan awal dan keadaan tujuan, serta menghasilkan node-node awal, sementara class `Node` digunakan untuk merepresentasikan setiap keadaan dalam permainan Eight Puzzle dan menyimpan informasi penting tentang keadaan tersebut, biayanya, dan node induknya.

# 3. Ubahlah initial dan goal state dari program di atas sehingga bentuk initial dan goal statenya Gambar 5.9. Kemudian tentukan langkah-langkah mana saja sehingga puzzlenya mencapai goal state. Analisa dan bedakan dengan solusi pada point 1 dan 2.
# Code:
    package JavaApplivation2;
    import java.util.Vector;

    class Node {
    int[] state = new int[9];
    int cost;
    Node parent = null;
    Vector<Node> successors = new Vector<Node>();

    Node(int s[], Node parent) {
        this.parent = parent;
        for (int i = 0; i < 9; i++)
            state[i] = s[i];
    }

    public Node() {
    }

    public String toString() {
        String s = "";
        for (int i = 0; i < 9; i++) {
            s = s + state[i] + " ";
        }
        return s;
    }

    public boolean equals(Object node) {
        Node n = (Node) node;
        boolean result = true;
        for (int i = 0; i < 9; i++) {
            if (n.state[i] != state[i])
                result = false;
        }
        return result;
    }

    Vector<Node> getPath(Vector<Node> v) {
        v.insertElementAt(this, 0);
        if (parent != null)
            v = parent.getPath(v);
        return v;
    }

    Vector<Node> getPath() {
        return getPath(new Vector<Node>());
    }
    }

    class EightPuzzleSpace {
    Node getRoot() {
        int initialState[] ={1, 5, 3, 4, 6, 8, 2, 7, 0};
        return new Node(initialState, null);
    }

    Node getGoal() {
        int goalState[] = {7, 6, 5, 8, 0, 4, 1, 2, 3};
        return new Node(goalState, null);
    }

    Vector<Node> getSuccessors(Node parent) {
        Vector<Node> successors = new Vector<Node>();
        for (int r = 0; r < 3; r++) {
            for (int c = 0; c < 3; c++) {
                if (parent.state[(r * 3) + c] == 0) {
                    if (r > 0) {
                        successors.add(transformState(r - 1, c, r, c, parent));
                    }
                    if (r < 2) {
                        successors.add(transformState(r + 1, c, r, c, parent));
                    }
                    if (c > 0) {
                        successors.add(transformState(r, c - 1, r, c, parent));
                    }
                    if (c < 2) {
                        successors.add(transformState(r, c + 1, r, c, parent));
                    }
                }
            }
        }
        parent.successors = successors;
        return successors;
    }

    Node transformState(int r0, int c0, int r1, int c1, Node parent) {
        int[] s = parent.state;
        int[] newState = { s[0], s[1], s[2], s[3], s[4], s[5], s[6], s[7], s[8] };
        newState[(r1 * 3) + c1] = s[(r0 * 3) + c0];
        newState[(r0 * 3) + c0] = 0;
        return new Node(newState, parent);
    }
    
    public static void main(String args[]) {
        new EightPuzzleSpace().run();
    }

    private void run() {
    }
    }

    class EightPuzzleSearch {
    EightPuzzleSpace space = new EightPuzzleSpace();
    Vector<Node> open = new Vector<Node>();
    Vector<Node> closed = new Vector<Node>();

    int h1Cost(Node node) {
        int cost = 0;
        for (int i = 0; i < node.state.length; i++) {
        if (node.state[i] != i) cost++; }
        return cost;
        }

    int h2Cost(Node node) {
        int cost = 0;
        int state[] = node.state;
        for (int i = 0; i < state.length; i++) {
            int v0 = i, v1 = state[i];
            if (v1 == 0)
                continue;
            int row0 = v0 / 3, col0 = v0 % 3, row1 = v1 / 3, col1 = v1 % 3;
            int c = (Math.abs(row0 - row1) + Math.abs(col0 - col1));
            cost += c;
        }
        return cost;
    }
    /*boleh diubah dengan memakai heuristic h1 atau h2 */
    int hCost(Node node) {
    return h1Cost(node);
    }
    Node getBestNode(Vector nodes) {
        int index = 0, minCost = Integer.MAX_VALUE;
        for (int i = 0; i < nodes.size(); i++) {
            Node node = (Node)nodes.elementAt(i);
            if (node.cost < minCost) {
                minCost = node.cost;
                index = i;
            }
        }
        Node bestNode = (Node)nodes.remove(index);
        return bestNode;
    }

    int getPreviousCost(Node node) {
        int i = open.indexOf(node);
        int cost = Integer.MAX_VALUE;
        if (i != -1) {
            cost = open.get(i).cost;
        } else {
            i = closed.indexOf(node);
            if (i != -1)
                cost = closed.get(i).cost;
        }
        return cost;
    }

    void printPath(Vector path) {
        for (int i = 0; i < path.size(); i++) {
            System.out.print(" " + path.elementAt(i) + "\n");
        }
    }

    void run() {
        Node root = space.getRoot();
        Node goal = space.getGoal();
        Node solution = null;
        open.add(root);
        System.out.print("\nRoot: " + root + "\n\n");
        while (open.size() > 0) {
            Node node = getBestNode(open);
            int pathLength = node.getPath().size();
            closed.add(node);
            if (node.equals(goal)) {
                solution = node;
                break;
            }
            Vector<Node> successors = space.getSuccessors(node);
            for (int i = 0; i < successors.size(); i++) {
                Node successor = successors.get(i);
                int cost = hCost(successor) + pathLength + 1;
                int previousCost;
                previousCost = getPreviousCost(successor);
                boolean inClosed;
                inClosed = closed.contains(successor);
                boolean inOpen = open.contains(successor);
                if (!(inClosed || inOpen) || cost < previousCost) {
                    if (inClosed)
                        closed.remove(successor);
                    if (!inOpen)
                        open.add(successor);
                    successor.cost = cost;
                    successor.parent = node;
                }
            }
        }
        if (solution != null) {
            Vector path = solution.getPath();
            System.out.print("\nSolution found\n");
            printPath(path);
        }
    }

    public static void main(String args[]) {
        new EightPuzzleSearch().run();
    }
    }

Code di atas menjelaskan tentang implementasi algoritma A* (A-star) dalam menyelesaikan masalah 8-puzzle. Dalam algoritma ini, terdapat dua heuristik yang dapat digunakan, yaitu h1Cost dan h2Cost, yang akan memengaruhi cara algoritma berjalan. Berikut adalah langkah-langkah yang perlu diikuti untuk mencapai keadaan tujuan dalam kode tersebut:

1. Kondisi Awal (Root):
    - Kondisi awal puzzle adalah {1, 5, 3, 4, 6, 8, 2, 7, 0}.
2. Kondisi Goal:
    - Kondisi tujuan yang harus dicapai adalah {7, 6, 5, 8, 0, 4, 1, 2, 3}.
3. Jalannya Algoritma:
    - Algoritma A* dimulai dengan kondisi awal (Root).
    - Algoritma memeriksa semua kemungkinan langkah dari kondisi awal, mempertimbangkan posisi kotak kosong, dan menggunakan salah satu heuristik (h1Cost atau h2Cost) untuk menghitung biaya.
    - Algoritma mencari jalur paling efisien menuju keadaan tujuan dengan mempertimbangkan biaya total (biaya heuristik, jarak yang sudah ditempuh, dan biaya sebelumnya).
    - Setiap langkah yang diambil untuk mencapai keadaan tujuan dipilih berdasarkan prioritas biaya terendah, yang ditentukan oleh heuristik yang digunakan.
    - Algoritma terus berjalan hingga mencapai keadaan tujuan.

Penggunaan heuristik h1Cost dan h2Cost akan memengaruhi perhitungan biaya dan cara algoritma berjalan. Kedua heuristik ini dapat menghasilkan solusi yang berbeda dalam beberapa kasus. Heuristik h1Cost menghitung biaya dengan menghitung jumlah angka yang tidak berada pada posisi yang benar, sementara h2Cost menghitung biaya berdasarkan jarak Manhattan antara angka yang salah dengan posisi yang benar.

# 4. Ubahlah initial dan goal state dari program di atas sehingga bentuk initial dan goal statenya Gambar 5.10. Kemudian tentukan langkah-langkah mana saja sehingga puzzlenya mencapai goal state. Analisa dan bedakan dengan solusi pada point 1, 2, dan 3.
#  Code:
    package JavaApplivation2;
    import java.util.Vector;
  
    class Node {
      int[] state = new int[9];
      int cost;
      Node parent = null;
      Vector<Node> successors = new Vector<Node>();
  
      Node(int s[], Node parent) {
          this.parent = parent;
          for (int i = 0; i < 9; i++)
              state[i] = s[i];
      }
  
      public Node() {
      }
  
      public String toString() {
          String s = "";
          for (int i = 0; i < 9; i++) {
              s = s + state[i] + " ";
          }
          return s;
      }
  
      public boolean equals(Object node) {
          Node n = (Node) node;
          boolean result = true;
          for (int i = 0; i < 9; i++) {
              if (n.state[i] != state[i])
                  result = false;
          }
          return result;
      }
  
      Vector<Node> getPath(Vector<Node> v) {
          v.insertElementAt(this, 0);
          if (parent != null)
              v = parent.getPath(v);
          return v;
      }
  
      Vector<Node> getPath() {
          return getPath(new Vector<Node>());
      }
      public static void main(String args[]) {
          new Node().run();
      }
  
      private void run() {
      }
    }

    class EightPuzzleSpace {
      Node getRoot() {
          int initialState[] ={1, 2, 3, 4, 5, 6, 7, 8, 0};
          return new Node(initialState, null);
      }
  
      Node getGoal() {
          int goalState[] = {1, 2, 3, 4, 0, 5, 6, 7, 8};
          return new Node(goalState, null);
      }
  
      Vector<Node> getSuccessors(Node parent) {
          Vector<Node> successors = new Vector<Node>();
          for (int r = 0; r < 3; r++) {
              for (int c = 0; c < 3; c++) {
                  if (parent.state[(r * 3) + c] == 0) {
                      if (r > 0) {
                          successors.add(transformState(r - 1, c, r, c, parent));
                      }
                      if (r < 2) {
                          successors.add(transformState(r + 1, c, r, c, parent));
                      }
                      if (c > 0) {
                          successors.add(transformState(r, c - 1, r, c, parent));
                      }
                      if (c < 2) {
                          successors.add(transformState(r, c + 1, r, c, parent));
                      }
                  }
              }
          }
          parent.successors = successors;
          return successors;
      }
  
      Node transformState(int r0, int c0, int r1, int c1, Node parent) {
          int[] s = parent.state;
          int[] newState = { s[0], s[1], s[2], s[3], s[4], s[5], s[6], s[7], s[8] };
          newState[(r1 * 3) + c1] = s[(r0 * 3) + c0];
          newState[(r0 * 3) + c0] = 0;
          return new Node(newState, parent);
      }
      
      public static void main(String args[]) {
          new EightPuzzleSpace().run();
      }
  
      private void run() {
      }
      
    }
  
    class EightPuzzleSearch {
      EightPuzzleSpace space = new EightPuzzleSpace();
      Vector<Node> open = new Vector<Node>();
      Vector<Node> closed = new Vector<Node>();
  
      int h1Cost(Node node) {
          int cost = 0;
          for (int i = 0; i < node.state.length; i++) {
          if (node.state[i] != i) cost++; }
          return cost;
          }
  
      int h2Cost(Node node) {
          int cost = 0;
          int state[] = node.state;
          for (int i = 0; i < state.length; i++) {
              int v0 = i, v1 = state[i];
              if (v1 == 0)
                  continue;
              int row0 = v0 / 3, col0 = v0 % 3, row1 = v1 / 3, col1 = v1 % 3;
              int c = (Math.abs(row0 - row1) + Math.abs(col0 - col1));
              cost += c;
          }
          return cost;
      }
    /*boleh diubah dengan memakai heuristic h1 atau h2 */
      int hCost(Node node) {
      return h1Cost(node);
      }
      Node getBestNode(Vector nodes) {
          int index = 0, minCost = Integer.MAX_VALUE;
          for (int i = 0; i < nodes.size(); i++) {
              Node node = (Node)nodes.elementAt(i);
              if (node.cost < minCost) {
                  minCost = node.cost;
                  index = i;
              }
          }
          Node bestNode = (Node)nodes.remove(index);
          return bestNode;
      }
  
      int getPreviousCost(Node node) {
          int i = open.indexOf(node);
          int cost = Integer.MAX_VALUE;
          if (i != -1) {
              cost = open.get(i).cost;
          } else {
              i = closed.indexOf(node);
              if (i != -1)
                  cost = closed.get(i).cost;
          }
          return cost;
      }
  
      void printPath(Vector path) {
          for (int i = 0; i < path.size(); i++) {
              System.out.print(" " + path.elementAt(i) + "\n");
          }
      }
  
      void run() {
          Node root = space.getRoot();
          Node goal = space.getGoal();
          Node solution = null;
          open.add(root);
          System.out.print("\nRoot: " + root + "\n\n");
          while (open.size() > 0) {
              Node node = getBestNode(open);
              int pathLength = node.getPath().size();
              closed.add(node);
              if (node.equals(goal)) {
                  solution = node;
                  break;
              }
              Vector<Node> successors = space.getSuccessors(node);
              for (int i = 0; i < successors.size(); i++) {
                  Node successor = successors.get(i);
                  int cost = hCost(successor) + pathLength + 1;
                  int previousCost;
                  previousCost = getPreviousCost(successor);
                  boolean inClosed;
                  inClosed = closed.contains(successor);
                  boolean inOpen = open.contains(successor);
                  if (!(inClosed || inOpen) || cost < previousCost) {
                      if (inClosed)
                          closed.remove(successor);
                      if (!inOpen)
                          open.add(successor);
                      successor.cost = cost;
                      successor.parent = node;
                  }
              }
          }
          if (solution != null) {
              Vector path = solution.getPath();
              System.out.print("\nSolution found\n");
              printPath(path);
          }
      }
  
      public static void main(String args[]) {
          new EightPuzzleSearch().run();
      }
    }

Code diatas mejelaskan tentang implementasi di "code 3" adalah penerapan algoritma A* untuk menyelesaikan masalah 8-puzzle. Dalam implementasi ini, ada dua pilihan heuristik, yaitu h1Cost dan h2Cost, yang mempengaruhi alur algoritma. Heuristik ini digunakan untuk menghitung biaya setiap langkah dalam pencarian solusi. Anda diminta untuk menentukan langkah-langkah yang harus diambil untuk mencapai keadaan akhir dan membandingkannya dengan solusi pada poin 1, code 1, dan code 2.
Berikut adalah langkah-langkah untuk menentukan solusi 8-puzzle dengan menggunakan implementasi di "code 3":

1. Kondisi Awal (Root):
   - {1, 2, 3, 4, 5, 6, 7, 8, 0}
2. Kondisi Goal:
   - {1, 2, 3, 4, 0, 5, 6, 7, 8}
3. Alur Algoritma:
   - Algoritma A* dimulai dengan kondisi awal (Root).
   - Algoritma mencari langkah-langkah untuk mencapai keadaan akhir (Goal).
   - Algoritma menggunakan salah satu heuristik, yaitu h1Cost atau h2Cost, untuk menghitung biaya setiap langkah.
   - Algoritma memilih langkah-langkah yang memiliki biaya terendah berdasarkan heuristik yang dipilih.
   - Algoritma terus berjalan hingga mencapai keadaan akhir (Goal).

Perbandingan dengan solusi pada poin 1, code 1, dan code 2:
- Solusi pada poin 1, code 1, dan code 2 juga menggunakan algoritma A* untuk menyelesaikan masalah 8-puzzle, namun mungkin menggunakan heuristik yang berbeda.
- Perbedaan heuristik yang digunakan dapat mempengaruhi urutan langkah-langkah dan waktu yang dibutuhkan untuk mencapai keadaan akhir (Goal).

Solusi dalam "code 3" di atas akan memberikan langkah-langkah spesifik yang mengarah ke keadaan akhir {1, 2, 3, 4, 0, 5, 6, 7, 8} berdasarkan heuristik yang dipilih (h1Cost atau h2Cost). Langkah-langkah tersebut dapat ditemukan dalam keluaran dari implementasi "code 3" setelah dijalankan.

# 5. Ubahlah initial dan goal state dari program dan class-class di atas sehingga bentuk initial dan goal statenya Gambar 5.11. Kemudian tentukan langkah-langkah mana saja sehingga puzzlenya mencapai goal state.
# Code:
    package JavaApplication2;
    import java.util.Vector;

    class Node {
    char[] state = new char[9];
    int cost;
    Node parent = null;
    Vector<Node> successors = new Vector<Node>();

    Node(char s[], Node parent) {
        this.parent = parent;
        for (int i = 0; i < 9; i++)
            state[i] = s[i];
    }

    public Node() {
    }

    public String toString() {
        String s = "";
        for (int i = 0; i < 9; i++) {
            s = s + state[i] + " ";
        }
        return s;
    }

    public boolean equals(Object node) {
        Node n = (Node) node;
        boolean result = true;
        for (int i = 0; i < 9; i++) {
            if (n.state[i] != state[i])
                result = false;
        }
        return result;
    }

    Vector<Node> getPath(Vector<Node> v) {
        v.insertElementAt(this, 0);
        if (parent != null)
            v = parent.getPath(v);
        return v;
    }

    Vector<Node> getPath() {
        return getPath(new Vector<Node>());
    }

    public static void main(String args[]) {
        new Node().run();
    }

    private void run() {
    }
    }

    class EightPuzzleSpace {
    Node getRoot() {
        char initialState[] = {'D', 'B', 'E', 'A', 'F', 'G', 'H', 'C', 'x'};
        return new Node(initialState, null);
    }

    Node getGoal() {
        char goalState[] = {'A', 'H', 'G', 'B', 'x', 'F', 'C', 'D', 'E'};
        return new Node(goalState, null);
    }

    Vector<Node> getSuccessors(Node parent) {
        Vector<Node> successors = new Vector<Node>();
        for (int r = 0; r < 3; r++) {
            for (int c = 0; c < 3; c++) {
                if (parent.state[(r * 3) + c] == 'x') {
                    if (r > 0) {
                        successors.add(transformState(r - 1, c, r, c, parent));
                    }
                    if (r < 2) {
                        successors.add(transformState(r + 1, c, r, c, parent));
                    }
                    if (c > 0) {
                        successors.add(transformState(r, c - 1, r, c, parent));
                    }
                    if (c < 2) {
                        successors.add(transformState(r, c + 1, r, c, parent));
                    }
                }
            }
        }
        parent.successors = successors;
        return successors;
    }

    Node transformState(int r0, int c0, int r1, int c1, Node parent) {
        char[] s = parent.state;
        char[] newState = {'x', 'x', 'x', 'x', 'x', 'x', 'x', 'x', 'x'};
        newState[(r1 * 3) + c1] = s[(r0 * 3) + c0];
        newState[(r0 * 3) + c0] = 'x';
        return new Node(newState, parent);
    }

    public static void main(String args[]) {
        new EightPuzzleSpace().run();
    }

    private void run() {
    }
    }

    class EightPuzzleSearch {
    EightPuzzleSpace space = new EightPuzzleSpace();
    Vector<Node> open = new Vector<Node>();
    Vector<Node> closed = new Vector<Node>();

    int h1Cost(Node node) {
        int cost = 0;
        for (int i = 0; i < node.state.length; i++) {
            if (node.state[i] != 'x')
                cost++;
        }
        return cost;
    }

    int h2Cost(Node node) {
        int cost = 0;
        char state[] = node.state;
        for (int i = 0; i < state.length; i++) {
            char v = state[i];
            if (v != 'x') {
                int row0 = i / 3, col0 = i % 3;
                int row1 = i / 3, col1 = i % 3;
                if (v != 'x') {
                    int c = (Math.abs(row0 - row1) + Math.abs(col0 - col1));
                    cost += c;
                }
            }
        }
        return cost;
    }

    int hCost(Node node) {
        return h1Cost(node);
    }

    Node getBestNode(Vector nodes) {
        int index = 0, minCost = Integer.MAX_VALUE;
        for (int i = 0; i < nodes.size(); i++) {
            Node node = (Node) nodes.elementAt(i);
            if (node.cost < minCost) {
                minCost = node.cost;
                index = i;
            }
        }
        Node bestNode = (Node) nodes.remove(index);
        return bestNode;
    }

    int getPreviousCost(Node node) {
        int i = open.indexOf(node);
        int cost = Integer.MAX_VALUE;
        if (i != -1) {
            cost = open.get(i).cost;
        } else {
            i = closed.indexOf(node);
            if (i != -1)
                cost = closed.get(i).cost;
        }
        return cost;
    }

    void printPath(Vector path) {
        for (int i = 0; i < path.size(); i++) {
            System.out.print(" " + path.elementAt(i) + "\n");
        }
    }

    void run() {
        Node root = space.getRoot();
        Node goal = space.getGoal();
        Node solution = null;
        open.add(root);
        System.out.print("\nRoot: " + root + "\n\n");
        while (open.size() > 0) {
            Node node = getBestNode(open);
            int pathLength = node.getPath().size();
            closed.add(node);
            if (node.equals(goal)) {
                solution = node;
                break;
            }
            Vector<Node> successors = space.getSuccessors(node);
            for (int i = 0; i < successors.size(); i++) {
                Node successor = successors.get(i);
                int cost = hCost(successor) + pathLength + 1;
                int previousCost;
                previousCost = getPreviousCost(successor);
                boolean inClosed;
                inClosed = closed.contains(successor);
                boolean inOpen = open.contains(successor);
                if (!(inClosed || inOpen) || cost < previousCost) {
                    if (inClosed)
                        closed.remove(successor);
                    if (!inOpen)
                        open.add(successor);
                    successor.cost = cost;
                    successor.parent = node;
                }
            }
        }
        if (solution != null) {
            Vector path = solution.getPath();
            System.out.print("\nSolution found\n");
            printPath(path);
        }
    }

    public static void main(String args[]) {
        new EightPuzzleSearch().run();
    }
    }


Dalam menentukan langkah-langkah mencapai keadaan akhir pada Code 4 untuk puzzle, maka dapat mengimplementasikan algoritma pencarian seperti A* Search dan memanfaatkan dua heuristik, yaitu h1Cost dan h2Cost yang telah ditentukan dalam kode. Berikut adalah langkah-langkah umum yang dapat Anda ikuti:

1. Mulailah dengan menginisialisasi node awal menggunakan keadaan awal dari puzzle.
2. Selanjutnya, persiapkan dua daftar: open list dan closed list. Letakkan node awal di dalam open list.
3. Lakukan langkah-langkah berikut selama open list masih memiliki elemen di dalamnya:
   a. Pilih node dengan biaya terendah dari open list (biaya f = biaya heuristik + panjang path).
   b. Jika node yang dipilih adalah keadaan akhir (goal state), maka itu berarti puzzle telah selesai. Anda dapat mengakhiri proses pencarian.
   c. Selanjutnya, hasilkan semua node turunan dari node saat ini.
   d. Untuk setiap node turunan, hitung biaya f (menggunakan salah satu dari h1Cost atau h2Cost) dan panjang jalur dari node awal hingga node saat ini.
   e. Jika node turunan sudah ada dalam open list dan biaya f yang baru lebih rendah, maka perbarui biaya dan orang tua (parent) dari node tersebut.
   f. Jika node turunan sudah ada dalam closed list dan biaya f yang baru lebih rendah, perbarui biaya dan orang tua dari node tersebut, dan pindahkan node dari closed list ke dalam open list.
   g. Jika node turunan belum ada dalam kedua daftar (open list dan closed list), tambahkan node tersebut ke dalam open list.
   h. Akhirnya, tandai node saat ini sebagai telah dieksplorasi dengan memindahkannya dari open list ke dalam closed list.

Dengan mengikuti langkah-langkah di atas, maka dapat mengimplementasikan algoritma A* Search untuk mencapai keadaan akhir dalam puzzle Code 4.
