# Summary
    """
        Check for node left between min and node's val, and check for node.right between node.right and max
        simple inorder traversal
    """
    def check(node: Optional[TreeNode], min, max):
        if not node:
            return True;

        if (node.left and node.left.val > node.val) or  (node.right and node.right.val < node.val):
            return False

        return check(node.left, min, node.val) and check(node.right, node.val, max)

    return check(root)
- [Introduction](./introduction.md)
- [Initializing the project](./initialize.md) 
- [Building database models](./database.md)
    - [Models as structs](./models.md)
    - [Creating a connection to our database](./database-connection.md)
- [Building Todo API](./init_api.md)
    - [API Errors](./api-errors.md)
    - [Authentication Handlers](./api-auth.md)   
