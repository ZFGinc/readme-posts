# Динамическая обводка объекта как в Voices Of The Void

<img src="./1_gif_demo.gif"/>

В Update делаем рейкаст в направлении камеры. 

```csharp

    private void Update()
    {
        if (Physics.Raycast(CameraTransform.position, CameraTransform.forward, out _hitInfo, _rayDistance))
        {
            Debug.DrawRay(CameraTransform.position, CameraTransform.forward * _rayDistance, Color.yellow);
            
            if (_hitInfo.transform.TryGetComponent(out InteractObject interactObject))
            {
                if (Distance > _hoverDistance)
                {
                    HideInteractMenu();
                    return;
                }
    
                if (_currentInteractObject != null)
                {
                    HideInteractMenu();
                }
    
                ShowInteractMenu(interactObject);
            }
            else
            {
                HideInteractMenu();
            }
        }
    }

```

При попадание луча на объект InteractObject вызывем метод включении интерфейса объекта

```C#
private void ShowInteractMenu(InteractObject interactObject)
{
    _currentInteractObject = interactObject;
    _currentInteractObject.IsHover = true;
    _currentInteractObject.ShowInteractMenu();
}
```

Далее перейдем к объекту InteractObject. Вызывается метод ShowInteractMenu(), который по факту преобразует свой объект из мировых координат в коордтината экрана.
```C#
public virtual void ShowInteractMenu()
{
    _rect = GUIAroundObejct.GuiRect3DObject(gameObject);
}
```

А дальше криво-косо-костыльно-похуистично пытаюсь отрисовать по полученным данным прямоугольник, надпись сверху и справа от объекта (по честному я хз как это правильно написать, чтобы не выглядило как какое-то заклинание).
```C#
private void OnGUI()
{
    if (IsHover) return;

    // Меняю размер шрифта
    GUI.skin.label.fontSize = 30;

    // Имя объекта
    // Подгоняю под нужные размеры
    Rect label = _rect;
    //label.x += _deltaSize;
    label.y -= (35 + _outlineSize);
    label.xMax = _rect.xMax + _displayName.Length * 20;
    label.height = 25 + _deltaSize + _outlineSize;
    label.width = _displayName.Length * 20;

    // Отрисовываю задний фон и текст
    GUI.DrawTexture(label, _textureBackground, ScaleMode.StretchToFill, true, 0f);
    GUI.Label(label, _displayName);

    // Текст справа
    // Делаю базовые настройки одного текста, который буду потом
    Rect funcLabel = _rect;
    funcLabel.x = _rect.xMax + _outlineSize;
    funcLabel.xMax = _rect.xMax + 300;
    funcLabel.height = 50;

    // Отрисовываю и смещаю вниз
    if (_objectInfo.CanHoldObject)
    {
        GUI.DrawTexture(funcLabel, _textureBackground, ScaleMode.StretchToFill, true, 0f);
        GUI.Label(funcLabel, _textHoldGUI);
        funcLabel.y += 50;
    }

    // Отрисовываю и смещаю вниз
    if (_objectInfo.CanGrabObject)
    {
        GUI.DrawTexture(funcLabel, _textureBackground, ScaleMode.StretchToFill, true, 0f);
        GUI.Label(funcLabel, _textGrabGUI);
        funcLabel.y += 50;
    }

    // Отрисовываю и смещаю вниз
    if (_objectInfo.CanUseObjects)
    {
        GUI.DrawTexture(funcLabel, _textureBackground, ScaleMode.StretchToFill, true, 0f);
        GUI.Label(funcLabel, _textUseGUI);
        funcLabel.y += 50;
    }

    // Прямоугольная обводка
    Rect topLine = _rect;
    Rect bottomLine = _rect;
    Rect leftLine = _rect;
    Rect rightLine = _rect;

    topLine.height = _outlineSize;
    topLine.width += _outlineSize;
    bottomLine.height = _outlineSize;
    bottomLine.width += _outlineSize;
    bottomLine.y += _rect.height;

    GUI.DrawTexture(topLine, _textureOutline, ScaleMode.StretchToFill, true, 1f);
    GUI.DrawTexture(bottomLine, _textureOutline, ScaleMode.StretchToFill, true, 1f);

    leftLine.width = _outlineSize;
    leftLine.height += _outlineSize;
    rightLine.width = _outlineSize;
    rightLine.height += _outlineSize;
    rightLine.x += _rect.width;

    GUI.DrawTexture(leftLine, _textureOutline, ScaleMode.StretchToFill, true, 1f);
    GUI.DrawTexture(rightLine, _textureOutline, ScaleMode.StretchToFill, true, 1f);

}
```

А вот и сам класс для преобразования координат из мировых в координаты экрана
```C#
public static class GUIAroundObejct
    {
        public static Rect GuiRect3DObject(GameObject go)
        {

            Vector3 center = go.GetComponent<Renderer>().bounds.center;
            Vector3 extents = go.GetComponent<Renderer>().bounds.extents;

            Vector2[] extentPoints = new Vector2[8]
            {
                WorldToGUIPoint(new Vector3(center.x-extents.x, center.y-extents.y, center.z-extents.z)),
                WorldToGUIPoint(new Vector3(center.x+extents.x, center.y-extents.y, center.z-extents.z)),
                WorldToGUIPoint(new Vector3(center.x-extents.x, center.y-extents.y, center.z+extents.z)),
                WorldToGUIPoint(new Vector3(center.x+extents.x, center.y-extents.y, center.z+extents.z)),
                WorldToGUIPoint(new Vector3(center.x-extents.x, center.y+extents.y, center.z-extents.z)),
                WorldToGUIPoint(new Vector3(center.x+extents.x, center.y+extents.y, center.z-extents.z)),
                WorldToGUIPoint(new Vector3(center.x-extents.x, center.y+extents.y, center.z+extents.z)),
                WorldToGUIPoint(new Vector3(center.x+extents.x, center.y+extents.y, center.z+extents.z))
            };

            Vector2 min = extentPoints[0];
            Vector2 max = extentPoints[0];

            foreach (Vector2 v in extentPoints)
            {
                min = Vector2.Min(min, v);
                max = Vector2.Max(max, v);
            }

            return new Rect(min.x, min.y, max.x - min.x, max.y - min.y);
        }

        public static Vector2 WorldToGUIPoint(Vector3 world)
        {
            Vector2 screenPoint = Camera.main.WorldToScreenPoint(world);
            screenPoint.y = (float)Screen.height - screenPoint.y;
            return screenPoint;
        }
    }
```
