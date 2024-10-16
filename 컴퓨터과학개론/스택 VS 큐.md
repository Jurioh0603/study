스택은 후입선출(LIFO) 또는 선입후출(FILO)의 방식으로 동작하여지는 자료구조이다. <br>
나중에 삽입된 데이터가 먼저 삭제되는 또는 먼저 삽입된 데이터가 나중에 삭제되는 구조이다. <br>
예를 들어 박스에 책을 정리할 때 먼저 넣었던 책을 가장 마지막에 빼는 방식이다. 큐는 선입선출(FIFO)의 방식으로 동작하여지는 자료구조이다. <br>
먼저 삽입된 데이터가 먼저 삭제되는 구조이다. 예를 들어 식당에서 먼저 줄을 선 사람이 먼저 밥을 받고 나가는 방식이다. <br>
스택은 한쪽에서 삽입과 삭제가 이루어진다. 가장 끝부분을 top이라고 부른다. <br>
LIFO 특성상 중간에서 데이터를 직접 삭제하거나 추가할 수 없으며 순차적으로 데이터에 접근해야 한다. <br>
주로 재귀호출에 많이 사용된다. <br>
큐는 한쪽에서 삽입이 이루어지고 반대 끝에서 삭제가 이루어진다. <br>
FIFO 특성상 먼저 들어온 데이터가 먼저 처리되기 때문에 마찬가지로 데이터를 중간에서 직접 삭제하거나 추가할 수 없다. <br>
주로 프로세스 스케줄링, 대기열 등에 자주 사용된다. <br>
스택에서 삽입은 push, 삭제는 pop이라고 하는데 새로운 데이터는 스택의 top에서 push 된다. <br>
또한 삭제 시 top에 있는 데이터가 pop 된다. <br>
큐에서 새로운 데이터는 rear에 삽입되고, front에서 데이터가 삭제된다. <br>
스택은 top에서 값을 push 또는 pop 하므로 가장 최근에 삽입된 데이터에만 접근할 수 있다. <br>
큐는 rear에서 값을 push하고 front에서 값을 pop 하므로 가장 먼저 삽입된 데이터에 접근할 수 있다. <br>
스택은 연속된 메모리 공간을 차지하지 않기 때문에 메모리 효율성이 좋은 편인 반면 큐는 배열로 구현되었기 때문에 연속된 메모리 공간을 차지하여 메모리 낭비가 발생할 수 있다. <br> 
이를 위해 연결 리스트로 구현된 원형 큐로 단점을 보완하였다. <br>
