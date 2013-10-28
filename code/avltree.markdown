---
title: avl平衡树源码
layout: post
key: 1a829442-66ad-4eec-aedf-39c5ffc1c202
tags:
  - 
---


对AVL平衡树的实现，主要在于实现对节点的插入和删除，方法是先用二叉查找树的插入和删除方法进行插入和删除，再对插入或者删除过的树进行维护，使之满足 AVL平衡树的性质。这里对二叉树的插入和删除以及 AVL平衡树的插入和删除都附带写了一下，本来尝写成迭代形式的。最后发现删除时需考虑的情况较多，写成迭代形式，代码凌乱且恶心。有点强迫症，还是写成递归，稍微清晰一点。具体的 [AVL平衡树](/2013/10/27/avltree.html)介绍在这里。


	#include "stdio.h"
	#include "stdlib.h"

	struct node
	{
		int key;
		int height;
		node *left;
		node *right;

		node (int);
	};

	node ::node (int key)
	{
		this->key = key;
		this->height = 1;
		this->left   = NULL;
		this->right  = NULL;
	}

	int getHeight (node *root)
	{
		return root == NULL ? 0 : root->height;
	}

	int max (int a, int b)
	{
		return a > b ? a : b;
	}

	node* rightRoate (node *root)
	{
		node *lchild = root->left;
		root->left = lchild->right;
		lchild->right = root;

		root->height   = max (getHeight (root->left), getHeight (root->right)) + 1;
		lchild->height = max (getHeight (lchild->left), getHeight (lchild->right)) + 1;
		return lchild;
	}

	node* leftRoate (node *root)
	{
		node *rchild = root->right;
		root->right = rchild->left;
		rchild->left = root;

		root->height   = max (getHeight (root->left), getHeight (root->right)) + 1;
		rchild->height = max (getHeight (rchild->left), getHeight (rchild->right)) + 1;
		return rchild;
	}

	node* previousOrNext (node *root)
	{
		if (root == NULL) {
			return NULL;
		}

		// get the previous of root
		node *ptr = root->left;
		while (ptr != NULL && ptr->right != NULL) {
			ptr = ptr->right;
		}

		// if root has a previous node
		if (ptr != NULL) {
			return ptr;
		}

		// get the next node of root
		ptr = root->right;
		while (ptr != NULL && ptr->left != NULL) {
			ptr = ptr->left;
		}

		// if ptr is not NULL, then root has a next node
		// if ptr is NULL, that's to say, the root is a left node
		return ptr;
	}

	node* insert (node *root, int key)
	{
		if (root == NULL) {
			return new node (key);
		}
		// if the key exist
		if (key == root->key) {
			return root;
		}
		// insert key in left subtree
		if (key < root->key) {
			root->left = insert (root->left, key);
		// insert key in right subtree
		} else {
			root->right = insert (root->right, key);
		}
		return root;
	}


	node *remove (node *root, int key)
	{
		// the key do not exist
		if (root == NULL) {
			return NULL;
		}

		// if the key is in left subtree
		if (key < root->key) {
			root->left = remove (root->left, key);
			return root;
		}

		// if the key is in right subtree
		if (key > root->key) {
			root->right = remove (root->right, key);
			return root;
		}

		// if the key is in current node.
		// get the previous or next of current node
		node *adj = previousOrNext (root);

		// if adj is NULL, that's to say, root is a leaf node
		if (adj == NULL) {
			delete root;
			return NULL;
		}

		// replace the key of root with adj's
		root->key = adj->key;

		// if adj is a previous node
		if (adj->key < key) {
			root->left = remove (root->left, adj->key);
			return root;
		}

		// if adj if a next node.
		root->right = remove (root->right, adj->key);
		return root;
	}

	node* avltree_insert (node *root, int key)
	{
		if (root == NULL) {
			return new node (key);
		}
		// if the key exist
		if (key == root->key) {
			return root;
		}
		// insert key in left subtree
		if (key < root->key) {
			root->left = avltree_insert (root->left, key);
		// insert key in right subtree
		} else {
			root->right = avltree_insert (root->right, key);
		}
		
		root->height = max (getHeight (root->left), getHeight (root->right)) + 1;
		int balance_factor = getHeight (root->left) - getHeight (root->right);

		switch (balance_factor) {
			case 2:
				// need to ajust the left subtree
				
				// left left case
				if (key < root->left->key) {
					root = rightRoate (root);
				// left right case
				} else {
					root->left = leftRoate (root->left);
					root = rightRoate (root);
				}
				break;
			case -2:
				// need to adjust the right subtree
				
				// right right case
				if (key > root->right->key) {
					root = leftRoate (root);
				// right left case
				} else {
					root->right = rightRoate (root->right);
					root = leftRoate (root);
				}
				break;
			default:
				break;
		}

		return root;
	}

	node* avltree_remove (node *root, int key)
	{
		// the key do not exist
		if (root == NULL) {
			return NULL;
		}

		if (key < root->key) {

			// if the key is in left subtree
			root->left = avltree_remove (root->left, key);
		} else if (key > root->key) {

			// if the key is in right subtree
			root->right = avltree_remove (root->right, key);
		} else {

			// if the key is in current node.
			// get the previous or next of current node
			node *adj = previousOrNext (root);

			// if adj is NULL, that's to say, root is a leaf node
			if (adj == NULL) {
				delete root;
				return NULL;
			}

			// replace the key of root with adj's
			root->key = adj->key;

			if (adj->key < key) {
				// if adj is a previous node
				root->left = avltree_remove (root->left, adj->key);
			} else {
				// if adj if a next node.
				root->right = avltree_remove (root->right, adj->key);
			}
		}

		root->height = max (getHeight (root->left), getHeight (root->right)) + 1;
		int balance_factor = getHeight (root->left) - getHeight (root->right);

		switch (balance_factor) {
			case 2:
				// need to ajust the left subtree
				
				// left left case
				if (getHeight (root->left->left) >= getHeight (root->left->right)) {
					root = rightRoate (root);
				// left right case
				} else {
					root->left = leftRoate (root->left);
					root = rightRoate (root);
				}
				break;
			case -2:
				// need to adjust the right subtree
				
				// right right case
				if (getHeight (root->right->right) >= getHeight (root->right->left)) {
					root = leftRoate (root);
				// right left case
				} else {
					root->right = rightRoate (root->right);
					root = leftRoate (root);
				}
				break;
			default:
				break;
		}

		return root;
	}

	void travese (node *root)
	{
		if (root == NULL) {
			return;
		}
		travese (root->left);
		printf("%d ", root->key);
		travese (root->right);
	}

	int main()
	{
		node *root;
		int elem[100];
		for (int e = 0; e < 80; e++) {
			elem[e] = rand () % 100;
			root = avltree_insert (root, elem[e]);
		}
		for (int e = 0; e < 80; e++) {
			root = avltree_remove (root, elem[e]);
		}
		return 0;
	}

####引用####
[AVL Tree | Set 1 (Insertion)](http://www.geeksforgeeks.org/avl-tree-set-1-insertion/) <br>
[AVL Tree | Set 1 (Deletion)](http://www.geeksforgeeks.org/avl-tree-set-2-deletion/)
