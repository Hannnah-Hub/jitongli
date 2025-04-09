天平系统  

使用说明：  
场景中有可以拖动的物体（商品），每个都可以放到“天平”的左右两侧；  
每个商品有自己设定的重量；  
当商品被放在天平的一侧时，记录其重量；  
比较两边的总重量，天平会缓慢向下倾斜重的一侧（实现一个位移动画）；  

  商品：  
  每一个物体（商品）都需要带有 Collider2D 和 Rigidbody2D；  
  天平盘子：左右两侧是两个带 Collider2D 的物体
  天平轴：一个 物体 控制它的旋转或位移；  
  拖动物体：通过鼠标拖拽，可以放到左或右的盘子上。  

  
1. 商品拖动脚本 DraggableItem.cs
   ```C#
   using UnityEngine;

public class DraggableItem : MonoBehaviour
{
    private Vector3 offset;
    private bool isDragging = false;

    void OnMouseDown()
    {
        offset = transform.position - Camera.main.ScreenToWorldPoint(Input.mousePosition);
        isDragging = true;
    }

    void OnMouseDrag()
    {
        if (isDragging)
        {
            Vector3 newPos = Camera.main.ScreenToWorldPoint(Input.mousePosition) + offset;
            newPos.z = 0;
            transform.position = newPos;
        }
    }

    void OnMouseUp()
    {
        isDragging = false;
    }
}
```
