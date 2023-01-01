## 实现一个二叉查找树
=== "java"

    ``` java
    // 实现一个二叉查找树，并且支持插入、删除、查找操作
    public class BinarySearchTree {
        private Node tree;

        public Node find(int data) {
            Node p = tree;
            while (p != null) {
                if (data < p.data)
                    p = p.left;
                else if (data > p.data)
                    p = p.right;
                else
                    return p;
            }
            return null;
        }

        public void insert(int data) {
            if (tree == null) {
                tree = new Node(data);
                return;
            }
            Node p = tree;
            while (p != null) {
                if (data > p.data) {
                    if (p.right == null) {
                        p.right = new Node(data);
                        return;
                    }
                    p = p.right;
                } else {
                    if (p.left == null) {
                        p.left = new Node(data);
                        return;
                    }
                    p = p.left;
                }
            }
        }

        public void delete(int data) {
            Node p = tree; // p指向要删除的节点，初始化指向根节点
            Node pp = null; // pp记录的是p的父节点
            while (p != null && p.data != data) {
                pp = p;
                if (data > p.data)
                    p = p.right;
                else
                    p = p.left;
            }

            if (p == null)
                return;
            // 要删除的节点有两个子节点
            if (p.left != null && p.right != null) { // 查找右子树中最小节点
                Node minP = p.right;
                Node minPP = p; // minPP表示minP的父节点
                while (minP.left != null) {
                    minPP = minP;
                    minP = minP.left;
                }
                p.data = minP.data; // 将minP的数据替换到p中
                p = minP; // 下面就变成了删除minP了
                pp = minPP;
            }

            // 删除节点是叶子节点或者仅有一个子节点
            Node child; // p的子节点
            if (p.left != null)
                child = p.left;
            else if (p.right != null)
                child = p.right;
            else
                child = null;

            if (pp == null)
                tree = child; // 删除的是根节点
            else if (pp.left == p)
                pp.left = child;
            else
                pp.right = child;

        }

        public Node findMin() {
            if (tree == null)
                return null;
            Node p = tree;
            while (p.left != null) {
                p = p.left;
            }
            return p;
        }

        public Node findMax() {
            if (tree == null)
                return null;
            Node p = tree;
            while (p.right != null) {
                p = p.right;
            }
            return p;
        }

            //查找某个节点的后继节点
        public Node successor(Node x){
            if(x.right!=null){
                return min(x.right);
            }
            Node y = x.parent;
            while(y!=null && x==y.right){
                x = y;
                y = y.parent;
            }
            return y;
        }

        //查找某个节点的前驱节点
        public Node predecessor(Node x){
            if(x.left!=null){
                return max(x.left);
            }
            Node y = x.parent;
            while(y!=null && x==y.left){
                x = y;
                y = y.parent;
            }
            return y;
        }

        // 二分搜索树的前序遍历
        public void preOrder() {
            preOrder(tree);
        }

        // 二分搜索树的中序遍历
        public void inOrder() {
            inOrder(tree);
        }

        // 二分搜索树的后序遍历
        public void postOrder() {
            postOrder(tree);
        }

        // 前序遍历:先打印这个节点，然后再打印它的左子树，最后打印它的右子树。
        public preOrder(Node node){
            if(node == null) return;
            System.out.println(node.data);
            preOrder(node.left);
            preOrder(node.right);
        }

        // 中序遍历:先打印它的左子树，然后再打印它本身，最后打印它的右子树。
        public inOrder(Node node){
            if(node == null) return;
            inOrder(node.left);
            System.out.println(node.data);
            inOrder(node.right);
        }

        // 后序遍历:先打印它的左子树，然后再打印它的右子树，最后打印这个节点本身。
        public postOrder(Node node){
            if(node == null) return;
            postOrder(node.left);
            postOrder(node.right);
            System.out.println(node.data);
        }

        // 二分搜索树的层序遍历
        public levelOrder(){
            LinkedList<Node> q = new LinkedList<>();
            q.add(tree);
            while(!q.isEmpty()){
                Node node = q.remove();
                System.out.println(node.data);
                if(node.left!=null){
                    q.add(node.left);
                }
                if(node.right!=null){
                    q.add(node.right);
                }
            }
        }

        //[翻转二叉树](https://leetcode.cn/problems/invert-binary-tree/)
        public Node invertTree(Node root) {
            if (root == null)
                return null;
            Node left = invertTree(root.left);
            Node right = invertTree(root.right);
            root.left = right;
            root.right = left;
            return root;
        }

        //[二叉树的最大深度](https://leetcode-cn.com/problems/maximum-depth-of-binary-tree/)
        public Node maxDepth(Node root) {
            if (root == null)
                return 0;
            int left = maxDepth(root.left);
            int right = maxDepth(root.right);
            return Math.max(left, right) + 1;
        }
        
        //[验证二叉查找树](https://leetcode-cn.com/problems/validate-binary-search-tree/)
        public boolean isValidBST(TreeNode root) {
            return isValidBST(root, Long.MIN_VALUE, Long.MAX_VALUE);
        }

        public boolean isValidBST(TreeNode node, long lower, long upper) {
            if (node == null) {
                return true;
            }
            if (node.val <= lower || node.val >= upper) {
                return false;
            }
            return isValidBST(node.left, lower, node.val) && isValidBST(node.right, node.val, upper);
        }

        public static class Node {
            private int data;
            public Node left;
            public Node right;

            public Node(int data) {
                this.data = data;
            }

        }

    }

    ```   