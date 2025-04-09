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
2.商品重量组件 ItemWeight.cs
```C#
using UnityEngine;

public class ItemWeight : MonoBehaviour
{
    public float weight = 1f; // 自定义重量
}
```
3.主控制脚本BalanceManager.cs

```C#
using System.Collections.Generic;
using UnityEngine;

public class BalanceManager : MonoBehaviour
{
    public Transform leftPlate;
    public Transform rightPlate;
    public Transform balanceBar;

    public Collider2D leftArea;
    public Collider2D rightArea;

    public float maxOffset = 0.5f; // 盘子最大上下移动距离
    public float moveSpeed = 2f;
    public float maxAngle = 15f;   // 杠杆最大旋转角度

    void Update()
    {
        List<ItemWeight> leftItems = GetItemsInArea(leftArea);
        List<ItemWeight> rightItems = GetItemsInArea(rightArea);

        float leftWeight = GetTotalWeight(leftItems);
        float rightWeight = GetTotalWeight(rightItems);

        // 计算权重差（范围限制到 -1 到 1）
        float totalWeight = leftWeight + rightWeight;
        float balanceRatio = totalWeight > 0 ? (rightWeight - leftWeight) / totalWeight : 0f;
    Debug.Log($"左盘重量: {leftWeight} | 右盘重量: {rightWeight}");
        

        // 控制盘子高度：重的那边下降，轻的那边上升
        float offset = balanceRatio * maxOffset;

        Vector3 leftTargetPos = new Vector3(leftPlate.localPosition.x, offset, leftPlate.localPosition.z);
        Vector3 rightTargetPos = new Vector3(rightPlate.localPosition.x, -offset, rightPlate.localPosition.z);

        leftPlate.localPosition = Vector3.Lerp(leftPlate.localPosition, leftTargetPos, Time.deltaTime * moveSpeed);
        rightPlate.localPosition = Vector3.Lerp(rightPlate.localPosition, rightTargetPos, Time.deltaTime * moveSpeed);

        // 控制中间杆旋转
        float angle = balanceRatio * maxAngle;
        Quaternion targetRotation = Quaternion.Euler(0, 0, -angle);
        balanceBar.localRotation = Quaternion.Lerp(balanceBar.localRotation, targetRotation, Time.deltaTime * moveSpeed);
    }

    List<ItemWeight> GetItemsInArea(Collider2D area)
    {
        List<ItemWeight> items = new List<ItemWeight>();
        foreach (ItemWeight item in FindObjectsOfType<ItemWeight>())
        {
            if (item.GetComponent<Collider2D>().IsTouching(area))
            {
                items.Add(item);
            }
        }
        return items;
    }

    float GetTotalWeight(List<ItemWeight> items)
    {
        float total = 0f;
        foreach (var item in items)
        {
            total += item.weight;
        }
        return total;
    }
}
```
