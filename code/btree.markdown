---
title: B树简单源码
layout: post
key: 33e668e9-2f27-445e-848d-aa1b1e4258e2
tags:
  - 
---


说实话，需要考虑的情况很多，写的很烦，写了大半天，调试了半天。只是测试了主函数中那个简单数据，具体的介绍在[B树简要介绍](/2013/10/25/btree.html)。

	#include "stdio.h"

	template <typename keyType = int>
	struct node
	{
		bool leaf;
		int keyNum;
		keyType *keyList;
		node **children;
		node (int, bool);
	};

	template <typename keyType>
	node<keyType> :: node (int leastChildNum, bool leaf)
	{
		this->keyNum = 0;
		this->leaf   = leaf;
		this->keyList   = new keyType[leastChildNum * 2 - 1];
		this->children  = new node*[leastChildNum * 2];
	}


	template <typename keyType = int>
	struct btree
	{
		node<keyType> *root;
		int leastChildNum;
		void (*func) (keyType);

		static void print (keyType);
		btree (int, void (*) (keyType) = NULL);
		void travese (node<keyType>*);
		void insert (node<keyType>*, keyType);
		void remove (node<keyType>*, keyType);
		node<keyType>* split (node<keyType>*);	
		void merge (node<keyType>*, node<keyType>*, keyType);
	};

	template <typename keyType>
	void btree<keyType> ::print (keyType key)
	{
		if (sizeof (keyType) == 1) {
			printf("%c ", key);
			return;
		}
		printf("%d ", key);
	}

	template <typename keyType>
	btree<keyType> ::btree (int leastChildNum, void (*func) (keyType))
	{
		this->root = NULL;
		this->leastChildNum = leastChildNum;
		this->func = func == NULL ? print : func;
	}

	template <typename keyType>
	void btree<keyType> ::travese (node<keyType> *root)
	{
		for (int e = 0; e < root->keyNum; e++) {
			if (root->leaf == false) {
				travese (root->children[e]);
			}
			func (root->keyList[e]);
		}
		if (root->leaf == false) {
			travese (root->children[root->keyNum]);
		}
	}

	template <typename keyType>
	node<keyType>* btree<keyType> ::split (node<keyType> *left)
	{
		node<keyType> *right = new node<keyType> (leastChildNum, left->leaf);
		right->keyNum = leastChildNum - 1;

		for (int e = 0; e < right->keyNum; e++) {
			right->keyList[e]  = left->keyList[e + leastChildNum];
			right->children[e] = left->children[e + leastChildNum];
		}
		right->children[right->keyNum] = left->children[leastChildNum * 2 - 1];
		left->keyNum = leastChildNum - 1;
		return right;
	}

	template <typename keyType>
	void btree<keyType> ::insert (node<keyType> *root, keyType key)
	{
		// if the tree is NULL, or the node in root is full, the height of tree increase by 1.
		if (root == NULL || (root == this->root && root->keyNum == leastChildNum * 2 - 1)) {
			this->root = new node<keyType> (leastChildNum, root == NULL);
			this->root->children[0] = root;
			root = this->root;
		}

		int p = root->keyNum - 1;
		// if current node is a leaf, insert the key. 
		if (root->leaf == true) {
			while (p >= 0 && root->keyList[p] > key) {
				root->keyList[p + 1] = root->keyList[p];
				p--;
			}
			root->keyList[p + 1] = key;
			root->keyNum++;
			return;
		}

		// if current node is a interal node.
		while (p >= 0 && root->keyList[p] > key) {
			p--;
		}
		// if next child node will be searched is full, split it.
		if (root->children[++p]->keyNum == leastChildNum * 2 - 1) {
			keyType mid = root->children[p]->keyList[leastChildNum - 1];
			node<keyType> *right = split (root->children[p]);

			root->children[root->keyNum + 1] = root->children[root->keyNum];
			for (int e = root->keyNum - 1; e >= p; e--) {
				root->keyList[e + 1]  = root->keyList[e];
				root->children[e + 1] = root->children[e];
			}
			root->keyNum++;
			root->keyList[p] = mid;
			root->children[p + 1] = right;
		}
		
		// if the node is splited, and the key in the right subtree.
		if (p < root->keyNum && root->keyList[p] < key) {
			p++;
		}
		insert (root->children[p], key);
	}

	template <typename keyType>
	void btree<keyType> ::merge (node<keyType> *left, node<keyType> *right, keyType mid)
	{
		left->keyList[leastChildNum - 1] = mid;
		for (int e = 0; e < leastChildNum - 1; e++) {
			left->keyList[e + leastChildNum]  = right->keyList[e];
			left->children[e + leastChildNum] = right->children[e];
		}
		left->children[leastChildNum * 2 - 1] = right->children[leastChildNum - 1];
		left->keyNum = 2 * leastChildNum - 1;
	}

	template <typename keyType>
	void btree<keyType> ::remove (node<keyType> *root, keyType key)
	{
		if (root == NULL) {
			return;
		}

		int p = 0;
		while (p < root->keyNum && key > root->keyList[p]) {
			p++;
		}
		
		// if current node is a leaf node.
		if (root->leaf == true) {
			if (p == root->keyNum || key != root->keyList[p]) {
				return;
			}
			while (p < root->keyNum - 1) {
				root->keyList[p] = root->keyList[p + 1];
				p++;
			}
			root->keyNum--;
			return;
		}

		// if the key locate in current node.
		if (p < root->keyNum && key == root->keyList[p]) {
			if (root->children[p]->keyNum >= leastChildNum) {
				root->keyList[p] = root->children[p]->keyList[root->children[p]->keyNum - 1];
				remove (root->children[p], root->keyList[p]);
				return;
			}
			if (root->children[p + 1]->keyNum >= leastChildNum) {
				root->keyList[p] = root->children[p + 1]->keyList[0];
				remove (root->children[p + 1], root->keyList[p]);
				return;
			}

			merge (root->children[p], root->children[p + 1], key);
			for (int e = p; e < root->keyNum - 1; e++) {
				root->keyList[e]  = root->keyList[e + 1];
				root->children[e + 1] = root->children[e + 2];
			}
			//root->children[root->keyNum - 1] = root->children[root->keyNum];
			root->keyNum--;
			if (root->keyNum == 0 && root == this->root) {
				this->root = root->children[p];
			}
			remove (root->children[p], key);
			return;
		}

		// if the key do not locate in current node.
		if (root->children[p]->keyNum >= leastChildNum) {
			remove (root->children[p], key);
			return;
		}

		node<keyType> *pthChild = root->children[p];
		// if left sibling of the pth node has more than leastChildNum - 1 nodes,
		// then borrow a node from it
		if (p > 0 && root->children[p - 1]->keyNum >= leastChildNum) {

			node<keyType> *leftSibling = root->children[p - 1];
			pthChild->children[pthChild->keyNum + 1] = pthChild->children[pthChild->keyNum];
			for (int e = pthChild->keyNum - 1; e >= 0; e--) {
				pthChild->keyList[e + 1]  = pthChild->keyList[e];
				pthChild->children[e + 1] = pthChild->children[e];
			}
			pthChild->keyNum++;
			pthChild->keyList[0]  = root->keyList[p - 1];
			pthChild->children[0] = leftSibling->children[leftSibling->keyNum];

			root->keyList[p - 1] = leftSibling->keyList[leftSibling->keyNum - 1];
			leftSibling->keyNum--;

			remove (root->children[p], key);
			return;
		}

		// if right sibling of pth node has more than leastChildNum - 1 nodes,
		// then borrow a node from it
		if (p < root->keyNum && root->children[p + 1]->keyNum >= leastChildNum) {

			node<keyType> *rightSibling = root->children[p + 1];
			pthChild->keyList[pthChild->keyNum] = root->keyList[p];
			pthChild->children[pthChild->keyNum + 1] = rightSibling->children[0];
			root->keyList[p] = rightSibling->keyList[0];
			pthChild->keyNum++;

			for (int e = 1; e < rightSibling->keyNum; e++) {
				rightSibling->keyList[e - 1]  = rightSibling->keyList[e];
				rightSibling->children[e - 1] = rightSibling->children[e];
			}
			rightSibling->children[rightSibling->keyNum - 1] = rightSibling->children[rightSibling->keyNum];
			rightSibling->keyNum--;

			remove (root->children[p], key);
			return;
		}

		// if both left and right siblings have only leastChildNum - 1 nodes,
		// then merge one of sibling with current node
		if (p > 0) {
			p = p - 1;
		}
		merge (root->children[p], root->children[p + 1], root->keyList[p]);
		for (int e = p; e < root->keyNum - 1; e++) {
			root->keyList[e] = root->keyList[e + 1];
			root->children[e + 1] = root->children[e + 2];
		}

		root->keyNum--;
		if (root->keyNum == 0 && root == this->root) {
			this->root = root->children[p];
		}
		remove (root->children[p], key);
		return;
	}

	int main ()
	{
		char array[] = {  
		'G', 'M', 'P', 'W', 'A', 'C', 'D', 'E', 'J', 'K',  
		'N', 'O', 'R', 'S', 'T', 'U', 'V', 'X', 'Y', 'F', 'L', 'Z' };
		btree<char> tree (3);
		for (int e = 0; e < 22; e++) {
			tree.insert (tree.root, array[e]);
		}
		for (int e = 21; e >= 0; e--) {
			tree.remove (tree.root, array[e]);
		}
		tree.travese (tree.root);
		return 0;
	}
