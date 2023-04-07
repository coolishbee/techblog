

For full documentation visit [mkdocs.org](https://www.mkdocs.org).

## GameObject.SendMessage

!!! example

    === "C#"

        ```c#
        using UnityEngine;
        using System.Collections;

        public class ExampleClass : MonoBehaviour {
            void ApplyDamage(float damage) {
                print(damage);
            }
            void Example() {
                gameObject.SendMessage("ApplyDamage", 5.0F);
            }
        }
        ```