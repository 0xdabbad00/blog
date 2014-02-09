---
layout: post
title: Merge Sort
categories: []
tags: []
status: publish
type: post
published: true
meta:
  _edit_lock: '1333595199'
  _edit_last: '2'
  _syntaxhighlighter_encoded: '1'
---
I sometimes want a simple merge sort to review.  This one is simple, runs, and shows where it would be optimized in the real world.

[sourcecode language="java"]
public class test {
	private static final int minMergeSortListSize = 32;

	public static void mergeSortArray(int[] a, int start, int end) {
		/*
		if ((end - start) &lt; minMergeSortListSize) {
			// Use insertion sort for small datasets.
			for (int i = start; i &lt; end; i++) {
				int j;
				int v = a[i];
				for (j = i - 1; j &gt;= 0; j--) {
					if (a[j] &lt;= v)
						break;
					a[j + 1] = a[j];
				}
				a[j + 1] = v;
			}
			return;
		}
	    */
		if (end - start &lt;= 1) return;

		// Divide and conquer
		int middle = (end - start) / 2 + start;
		mergeSortArray(a, start, middle);
		mergeSortArray(a, middle, end);
		
		// Merge
		int[] temp = new int[end-start];
		int i1 = start;
		int i2 = middle;
		int tempi = 0;

		while (i1 &lt; middle &amp;&amp; i2 &lt; end) {
			if (a[i1] &lt;= a[i2])
				temp[tempi++] = a[i1++];
			else
				temp[tempi++] = a[i2++];
		}

		while (i1 &lt; middle) {
			temp[tempi++] = a[i1++];
		}
		while (i2 &lt; end) {
			temp[tempi++] = a[i2++];
		}
		System.arraycopy(temp, 0, a, start, end - start);
	}

	public static void disp(int[] a) {
		for (int tmp : a) {
			System.out.print(tmp + &quot; &quot;);
		}
		System.out.println();
	}

	public static void main(String[] argv) {
		int[] a = { 5, 1, 3, 4, 2, 7, 9, 6 };
	
		mergeSortArray(a, 0, a.length);
		disp(a);
	}
}
[/sourcecode] 
